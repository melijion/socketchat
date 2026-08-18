# Robot Control Station

## Description

This project is a **Robot Control Station** written in C++.

The station communicates with two robots using:

- **TCP** → sending commands to the robots.
- **UDP** → receiving information from the robots.
- **Threads** → allowing TCP communication and UDP communication to run separately.

The station is designed to control two robots connected to the same network.

---

## Communication

### TCP Communication

TCP is used by the control station to send commands to the robots.

The station uses:

```text
IP Address: 192.168.100.34
Port: 5000
```

The station works as a TCP server.
It waits for two robots to connect:

```text
Robot 1 connected!
Robot 2 connected!
```

Each robot gets its own socket:

```cpp
robotSocket[0] = robotSocket1;
robotSocket[1] = robotSocket2;
```

The selected robot is accessed using:

```cpp
robotSocket[id-1]
```

### UDP Communication

UDP is used by the robots to send their information back to the station.
The station listens on:

```text
IP Address: 192.168.100.34
Port: 5001
```

The robot information is received using:

```cpp
recvfrom()
```

The received information is then displayed in the console.

### Robot Information

The robots send their information using the following format:

```text
id=1;x=4.000000;y=5.000000;velocity=7.000000;state=error
```

The information contains:

| Information | Description |
| :--- | :--- |
| `id` | Robot ID |
| `x` | Robot X position |
| `y` | Robot Y position |
| `velocity` | Robot velocity |
| `state` | Current robot state |

---

## Threads

The station uses two threads.

### Command Thread

The first thread runs:

```cpp
sendcommand()
```

It is responsible for:

- Creating the TCP socket.
- Binding the socket.
- Listening for robots.
- Accepting the two robot connections.
- Displaying the menu.
- Asking the user for a robot ID.
- Asking for a command.
- Sending the command to the selected robot.

### Information Thread

The second thread runs:

```cpp
recvrobotinfo()
```

It is responsible for:

- Creating the UDP socket.
- Binding it to port 5001.
- Waiting for the `sendInfo` flag.
- Receiving robot information.
- Displaying the received information.

The threads are created in `main()`:

```cpp
thread sen(sendcommand, ref(sendInfo));
thread rcv(recvrobotinfo, ref(sendInfo));
```

---

## Main Menu

The station displays:

```text
=============MENU=============
1-enter command
2-exit
```

### Option 1 — Enter Command

The user first enters the robot ID:

```text
enter the robot id
```

Then the user enters the command:

```text
enter the command
```

Example:

```text
enter the robot id 1
enter the command demarrer
```

The command is then sent to Robot 1.

### Option 2 — Exit

When the user chooses:

```text
2
```

the station sends:

```text
done
```

to both robots and finishes the communication.

---

## Commands

The robot program currently handles commands such as:

- `demarrer`
- `arreter`
- `changer la vitesse`

### `demarrer`

Changes the robot state to:

```text
EN-MARCHE
```

### `arreter`

Changes the robot state to:

```text
ARRET
```

### `changer la vitesse`

Changes the robot velocity to:

```text
30
```

---

## Network Structure

The communication between the station and the robots can be represented as:

```text
                    CONTROL STATION
                    192.168.100.34
                           |
              +------------+------------+
              |                         |
           TCP :5000                UDP :5001
              |                         ^
              |                         |
        +-----+-----+             +-----+-----+
        |           |             |           |
     Robot 1     Robot 2       Robot 1     Robot 2
```

More simply:

```text
Station  ───── TCP ─────>  Robot
Station  <──── UDP ─────  Robot
```

- TCP is used for sending commands.
- UDP is used for receiving robot information.

---

## Program Structure

The station contains two main communication functions.

### `sendcommand()`

```cpp
void sendcommand(bool &sendInfo)
```

This function manages the TCP server, accepts the robot connections, displays the menu, and sends commands to the selected robot.

### `recvrobotinfo()`

```cpp
void recvrobotinfo(bool &sendInfo)
```

This function manages the UDP socket and receives robot information.

### `main()`

```cpp
int main()
```

The `main()` function:

- Starts Winsock.
- Displays the station information.
- Creates the command thread.
- Creates the robot information thread.
- Waits for both threads using `join()`.

---

## Libraries Used

The project uses the following C++ libraries:

```cpp
#include <iostream>
#include <thread>
#include <chrono>
#include <winsock2.h>
#include <ws2tcpip.h>
#include <windows.h>
#include <cstring>
#include <functional>
#include <string>
```

The Windows Winsock library is linked using:

```cpp
#pragma comment(lib, "ws2_32.lib")
```

---

## Starting the Program

When the program starts, it displays:

```text
Control Station starting...
=================================
     ROBOT CONTROL STATION
=================================
```

Winsock is initialized using:

```cpp
WSAStartup(MAKEWORD(2, 2), &wsaData);
```

The station then starts the TCP and UDP communication threads.
The station waits for the two robots to connect before displaying the command menu.

---

## Communication Flow

The basic communication process is:

1. Start Control Station
2. Start TCP server
3. Robot 1 connects
4. Robot 2 connects
5. User selects a robot
6. User enters a command
7. Command is sent using TCP
8. Robot processes the command
9. Robot sends its information using UDP
10. Station receives and displays the information

---

## Technologies Used

- C++
- Windows Sockets (Winsock2)
- TCP
- UDP
- Multithreading (`std::thread`, `std::string`, `std::chrono`)

---

## Current Limitations

This is the current version of the project and focuses mainly on implementing TCP/UDP communication and multithreading.
Some parts can still be improved, including:

- More detailed socket error handling.
- Better synchronization between threads.
- Validation of the robot ID entered by the user.
- Handling robot disconnections.
- Closing sockets properly when communication finishes.
- Better handling of TCP messages.
- Better association between a sent command and the corresponding UDP information.

---

## Project Goal

The goal of this project is to create a control station capable of communicating with multiple robots through a network.
The control station sends commands to the robots using TCP and receives their information using UDP, while using separate threads to handle the two communication processes.
