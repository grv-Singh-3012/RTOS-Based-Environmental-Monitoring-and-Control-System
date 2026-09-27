# RTOS-Based Environmental Monitoring and Control System

A real-time embedded environmental monitoring and control system developed using **STM32**, **FreeRTOS**, and **Embedded C**.

The system uses multiple FreeRTOS tasks for **sensor acquisition, data processing, UART communication, and fault monitoring**. FreeRTOS queues and mutexes are used for reliable inter-task communication and resource sharing, while interrupt-driven event handling improves system responsiveness.

---

## Project Overview

Environmental monitoring systems need to continuously acquire sensor data, process measurements, detect abnormal conditions, and communicate system status.

Instead of implementing all operations inside a single execution loop, this project uses **FreeRTOS** to divide the application into multiple independent tasks.

The main tasks are:

- Sensor Acquisition Task
- Data Processing Task
- UART Communication Task
- Fault Monitoring Task

This architecture improves modularity, responsiveness, and maintainability of the embedded application.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| **C** | Embedded application development |
| **FreeRTOS** | Real-time task scheduling and synchronization |
| **STM32** | Microcontroller platform |
| **UART** | Serial communication and debugging |
| **I2C** | Sensor communication |
| **GPIO** | Peripheral interfacing and control |
| **Queues** | Inter-task data communication |
| **Mutexes** | Shared-resource synchronization |
| **Interrupts** | Event-driven processing |

---

## Key Features

- Real-time environmental sensor monitoring
- Multi-tasking using FreeRTOS
- Independent sensor acquisition task
- Dedicated data processing task
- UART-based communication and debugging
- Fault monitoring
- I2C sensor interfacing
- GPIO peripheral control
- Queue-based inter-task communication
- Mutex-based resource protection
- Interrupt-driven event handling
- Task priority optimization
- Memory usage optimization
- Improved real-time response

---

## System Architecture

```text
                  +----------------------+
                  | Environmental Sensors|
                  +----------+-----------+
                             |
                         I2C / GPIO
                             |
                             v
              +-----------------------------+
              |       STM32 + FreeRTOS      |
              |                             |
              | +-------------------------+ |
              | | Sensor Acquisition Task | |
              | +------------+------------+ |
              |              |              |
              |              | Queue        |
              |              v              |
              | +-------------------------+ |
              | |  Data Processing Task   | |
              | +-----------+-------------+ |
              |             |               |
              |       +-----+------+        |
              |       |            |        |
              |       v            v        |
              | +-----------+ +-----------+ |
              | | UART Task | |Fault Task | |
              | +-----+-----+ +-----------+ |
              +-------|---------------------+
                      |
                    UART
                      |
                      v
              +----------------+
              | Serial Terminal|
              | / Debug Logs   |
              +----------------+
```

---

# FreeRTOS Task Design

## 1. Sensor Acquisition Task

The **Sensor Acquisition Task** periodically collects data from the connected environmental sensors.

Sensor modules communicate with the STM32 microcontroller through interfaces such as:

- I2C
- GPIO

After acquiring the sensor measurements, the task sends the data to the **Data Processing Task** using a FreeRTOS queue.

### Basic Flow

```text
Sensor
   |
   v
I2C / GPIO
   |
   v
Sensor Acquisition Task
   |
   | Queue
   v
Data Processing Task
```

This design prevents sensor acquisition from being tightly coupled with data processing.

---

## 2. Data Processing Task

The **Data Processing Task** receives sensor measurements from the FreeRTOS queue.

Its responsibilities include:

- Receiving sensor measurements
- Processing acquired data
- Preparing data for communication
- Supporting monitoring/control decisions
- Forwarding relevant information to other tasks

Separating processing from acquisition allows the FreeRTOS scheduler to manage both activities independently.

---

## 3. UART Communication Task

The **UART Communication Task** manages serial communication between the STM32 system and an external terminal.

UART is also used for debugging and monitoring the application during runtime.

Example output could follow a format such as:

```text
--------------------------------
Environmental Monitoring System
--------------------------------

Sensor Data Received

Temperature : <value>
System      : Running
Fault       : None
```

The exact displayed measurements depend on the sensors used in the implementation.

---

## 4. Fault Monitoring Task

The **Fault Monitoring Task** continuously supervises system conditions.

It can be used to identify conditions such as:

```text
Sensor communication failure
        |
        v
Invalid sensor reading
        |
        v
Fault Monitoring Task
        |
        v
Fault indication / UART log
```

The exact fault thresholds and recovery actions depend on the final hardware and application requirements.

---

# Inter-Task Communication

FreeRTOS **queues** are used to exchange data between tasks.

Example:

```text
Sensor Acquisition Task
        |
        |
        v
+------------------+
|  FreeRTOS Queue  |
+------------------+
        |
        |
        v
Data Processing Task
```

A simplified implementation can look like:

```c
typedef struct
{
    float temperature;
    float sensor_value;
} SensorData_t;

QueueHandle_t sensorQueue;
```

Queue creation:

```c
sensorQueue = xQueueCreate(10, sizeof(SensorData_t));
```

Sending data:

```c
xQueueSend(sensorQueue, &sensorData, portMAX_DELAY);
```

Receiving data:

```c
xQueueReceive(sensorQueue, &sensorData, portMAX_DELAY);
```

> The code above illustrates the intended FreeRTOS queue pattern. Adapt structure fields and queue size to the actual implementation.

---

# Mutex-Based Resource Protection

When multiple tasks access the same hardware resource, simultaneous access can create race conditions.

FreeRTOS mutexes can protect shared resources.

```c
SemaphoreHandle_t uartMutex;

uartMutex = xSemaphoreCreateMutex();
```

Example:

```c
if (xSemaphoreTake(uartMutex, portMAX_DELAY) == pdTRUE)
{
    /* Access shared UART resource */

    xSemaphoreGive(uartMutex);
}
```

This ensures that only one task accesses the protected resource at a time.

---

# Interrupt Handling

The system uses interrupt-driven event handling for time-sensitive hardware events.

The general design is:

```text
Hardware Event
      |
      v
Interrupt
      |
      v
Interrupt Service Routine
      |
      v
Notify / Signal FreeRTOS Task
      |
      v
Task Processes Event
```

Interrupt Service Routines should remain short.

Complex processing should normally be deferred to FreeRTOS tasks instead of being performed directly inside the interrupt handler.

---

# System Execution Flow

```text
Power ON
   |
   v
STM32 Initialization
   |
   v
GPIO Initialization
   |
   v
I2C Initialization
   |
   v
UART Initialization
   |
   v
Create Queues / Mutexes
   |
   v
Create FreeRTOS Tasks
   |
   v
Start Scheduler
   |
   +-----------------------+
   |                       |
   v                       v
Acquire Sensors       Monitor Faults
   |
   v
Queue Sensor Data
   |
   v
Process Data
   |
   v
UART Communication
   |
   v
Repeat
```

---

# Project Structure

A typical project structure can be organized as:

```text
RTOS-Environmental-Monitoring/
│
├── Core/
│   ├── Inc/
│   │   ├── main.h
│   │   ├── sensor.h
│   │   └── freertos.h
│   │
│   └── Src/
│       ├── main.c
│       ├── sensor.c
│       └── freertos.c
│
├── Drivers/
│
├── Middlewares/
│   └── FreeRTOS/
│
├── docs/
│   └── RTOS_Environmental_Monitoring_Project_Report.pdf
│
└── README.md
```

> The exact directory structure depends on the STM32 development environment and your actual repository.

---

# Task Communication

The overall communication flow is:

```text
                  Sensor Data
                      |
                      v
            +--------------------+
            | Sensor Acquisition |
            |       Task         |
            +---------+----------+
                      |
                    Queue
                      |
                      v
            +--------------------+
            | Data Processing    |
            |       Task         |
            +---------+----------+
                      |
              +-------+-------+
              |               |
              v               v
        +-----------+    +-----------+
        | UART Task |    |Fault Task |
        +-----------+    +-----------+
```

---

# Debugging

UART logging is used to monitor system behavior during development.

Debug information can include:

```text
Task started
Sensor data received
Queue message transmitted
Queue message received
UART transmission complete
Fault detected
Interrupt triggered
```

UART logging helps identify:

- Sensor communication problems
- Incorrect task execution
- Queue communication issues
- Timing problems
- Fault conditions

---

# Testing

The project can be tested at multiple levels.

| Test | Purpose |
|---|---|
| Sensor Test | Verify sensor acquisition |
| I2C Test | Verify sensor communication |
| GPIO Test | Verify digital input/output |
| UART Test | Verify serial communication |
| Queue Test | Verify inter-task communication |
| Mutex Test | Verify resource protection |
| Interrupt Test | Verify event handling |
| Fault Test | Verify fault monitoring |
| RTOS Task Test | Verify task scheduling |

---

# Performance Optimization

The application design considers optimization of:

### Task Priorities

Tasks are assigned priorities according to their timing requirements.

Time-sensitive tasks should receive appropriate scheduling priority without unnecessarily starving lower-priority tasks.

### Memory Usage

FreeRTOS task stacks, queues, and synchronization objects consume microcontroller RAM.

Memory usage should therefore be monitored and optimized.

### Response Time

Response time can be improved by:

- Keeping ISRs short
- Avoiding unnecessary blocking
- Selecting appropriate task priorities
- Reducing unnecessary processing
- Using queues efficiently
- Minimizing long critical sections

---

# Expected System Behavior

The system is designed to continuously:

1. Acquire environmental sensor data.
2. Transfer measurements through FreeRTOS queues.
3. Process the acquired data.
4. Monitor system conditions.
5. Report information through UART.
6. Respond to interrupt-driven events.
7. Continue real-time monitoring.

---

# Future Improvements

Possible improvements include:

- Additional environmental sensors
- Humidity monitoring
- Air-quality monitoring
- Pressure monitoring
- LCD/OLED display
- Wi-Fi connectivity
- Bluetooth connectivity
- IoT dashboard integration
- Cloud data logging
- Configurable alert thresholds
- Watchdog-based fault recovery
- Persistent data storage
- Remote system monitoring

---

# Skills Demonstrated

This project demonstrates practical experience with:

- Embedded C
- STM32 development
- FreeRTOS
- Real-time operating systems
- Task scheduling
- Multitasking
- Queues
- Mutexes
- Interrupt handling
- UART
- I2C
- GPIO
- Sensor interfacing
- Embedded debugging
- Memory optimization
- Real-time system design

---

# Documentation

Detailed project documentation can be placed inside:

```text
docs/
└── RTOS_Environmental_Monitoring_Project_Report.pdf
```

---

# Author

**Gaurav Singh**

B.Tech in Electrical Engineering  
Indian Institute of Technology Patna (IIT Patna)
