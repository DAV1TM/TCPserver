# Simple Winsock TCP Server and Client

## Overview

This project demonstrates a basic TCP server and client using Winsock in C++. The server listens for incoming connections and echoes messages back to the client. The client connects to the server, sends a message, and waits for the server's response.

## Files

- `server.cpp`: Contains the implementation of the TCP server.
- `client.cpp`: Contains the implementation of the TCP client.

## Prerequisites

- Windows operating system
- Visual Studio or any other C++ compiler that supports Winsock
- Basic knowledge of C++ and networking concepts

## Setup

### Server

1. **Initialize Winsock**: Before any Winsock function calls, the server initializes Winsock.
2. **Create a Socket**: A socket is created to listen for incoming connections.
3. **Bind the Socket**: The socket is bound to an IP address and port.
4. **Listen for Connections**: The socket is put into listening mode to accept incoming connections.
5. **Accept a Connection**: The server waits for a client to connect.
6. **Echo Messages**: The server echoes received messages back to the client.
7. **Cleanup**: The server closes the socket and cleans up Winsock.

### Client

1. **Initialize Winsock**: The client initializes Winsock before making any Winsock function calls.
2. **Create a Socket**: A socket is created to connect to the server.
3. **Connect to the Server**: The client connects to the server using the server's IP address and port.
4. **Send and Receive Data**: The client sends a message to the server and waits for the server's response.
5. **Cleanup**: The client closes the socket and cleans up Winsock.

## Usage

### Running the Server

1. Compile `server.cpp` using your C++ compiler.
2. Run the compiled server executable.
3. The server will start and wait for a client to connect.

### Running the Client

1. Compile `client.cpp` using your C++ compiler.
2. Run the compiled client executable.
3. The client will connect to the server, send a message, and print the server's response.

## Code Explanation

### Server (`server.cpp`)

```cpp
#include <iostream>
#include <WS2tcpip.h>
#pragma comment (lib, "ws2_32.lib")

using namespace std;

void main() {
    // Initialize Winsock
    WSADATA wsData;
    WORD ver = MAKEWORD(2, 2);
    int wsOK = WSAStartup(ver, &wsData);
    if (wsOK != 0) {
        cerr << "Can't Initialize Winsock! Quitting" << endl;
        return;
    }

    // Create a socket
    SOCKET listening = socket(AF_INET, SOCK_STREAM, 0);
    if (listening == INVALID_SOCKET) {
        cerr << "Can't create a socket! Quitting" << endl;
        return;
    }

    // Bind the IP address and port to a socket
    sockaddr_in hint;
    hint.sin_family = AF_INET;
    hint.sin_port = htons(54000);
    hint.sin_addr.S_un.S_addr = INADDR_ANY;
    bind(listening, (sockaddr*)&hint, sizeof(hint));

    // Tell Winsock the socket is for listening
    listen(listening, SOMAXCONN);

    // Wait for a connection
    sockaddr_in client;
    int clientSize = sizeof(client);
    SOCKET clientSocket = accept(listening, (sockaddr*)&client, &clientSize);
    if (clientSocket == INVALID_SOCKET) {
        cerr << "Can't connect to a socket! Quitting" << endl;
        return;
    }

    char host[NI_MAXHOST];
    char service[NI_MAXSERV];
    ZeroMemory(host, NI_MAXHOST);
    ZeroMemory(service, NI_MAXSERV);
    if (getnameinfo((sockaddr*)&client, sizeof(client), host, NI_MAXHOST, service, NI_MAXSERV, 0) == 0) {
        cout << host << " connected on port " << service << endl;
    } else {
        inet_ntop(AF_INET, &client.sin_addr, host, NI_MAXHOST);
        cout << host << " connected on port " << ntohs(client.sin_port) << endl;
    }

    // Close listening socket
    closesocket(listening);

    // While loop: accept and echo message back to client
    char buf[4096];
    while (true) {
        ZeroMemory(buf, 4096);
        int bytesReceived = recv(clientSocket, buf, 4096, 0);
        if (bytesReceived == SOCKET_ERROR) {
            cerr << "Error in recv()! Quitting" << endl;
            break;
        }
        if (bytesReceived == 0) {
            cout << "Client disconnected" << endl;
            break;
        }
        send(clientSocket, buf, bytesReceived + 1, 0);
    }

    // Close the socket
    closesocket(clientSocket);
    // Cleanup Winsock
    WSACleanup();
}
```

### Client (`client.cpp`)

```cpp
#include <iostream>
#include <WS2tcpip.h>
#pragma comment (lib, "ws2_32.lib")

using namespace std;

int main() {
    WSADATA wsData;
    WORD ver = MAKEWORD(2, 2);
    int wsOK = WSAStartup(ver, &wsData);
    if (wsOK != 0) {
        cerr << "Can't initialize Winsock! Quitting" << endl;
        return 1;
    }

    SOCKET sock = socket(AF_INET, SOCK_STREAM, 0);
    if (sock == INVALID_SOCKET) {
        cerr << "Can't create socket! Quitting" << endl;
        WSACleanup();
        return 1;
    }

    sockaddr_in hint;
    hint.sin_family = AF_INET;
    hint.sin_port = htons(55000);
    inet_pton(AF_INET, "127.0.0.1", &hint.sin_addr);

    int connResult = connect(sock, (sockaddr*)&hint, sizeof(hint));
    if (connResult == SOCKET_ERROR) {
        cerr << "Can't connect to server! Quitting" << endl;
        closesocket(sock);
        WSACleanup();
        return 1;
    }

    string userInput = "Hello from client";
    int sendResult = send(sock, userInput.c_str(), userInput.size() + 1, 0);

    if (sendResult != SOCKET_ERROR) {
        char buf[4096];
        ZeroMemory(buf, 4096);
        int bytesReceived = recv(sock, buf, 4096, 0);
        if (bytesReceived > 0) {
            cout << "Server: " << string(buf, 0, bytesReceived) << endl;
        }
    }

    closesocket(sock);
    WSACleanup();
    return 0;
}
```

## Troubleshooting

- Ensure that the server is running before starting the client.
- Verify that the IP address and port number in the client match the server.
- Check your firewall settings to ensure they are not blocking the connections.

## License

This project is licensed under the [MIT License](LICENSE). Feel free to use and modify the code as needed.

