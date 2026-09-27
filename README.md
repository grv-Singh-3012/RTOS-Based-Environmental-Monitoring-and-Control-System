# RTOS-Based-Environmental-Monitoring-and-Control-System
Developed a real-time embedded monitoring system on an STM32 microcontroller using FreeRTOS, integrating temperature and sensor modules through I2C and GPIO interfaces.  Implemented separate RTOS tasks for sensor acquisition, data processing, UART communication, and fault monitoring, using queues and mutexes for reliable inter-task communication .
                  +----------------------+
                  |   Sensor Modules     |
                  +----------+-----------+
                             |
                         I2C / GPIO
                             |
                             v
+-------------------------------------------------------+
|                 STM32 + FreeRTOS                      |
|                                                       |
|  +----------------+       +-----------------------+   |
|  | Sensor         | ----> | Data Processing Task  |   |
|  | Acquisition    | Queue |                       |   |
|  | Task           |       +-----------+-----------+   |
|  +----------------+                   |               |
|                                       |               |
|                         +-------------+-------------+ |
|                         |                           | |
|                         v                           v |
|                +----------------+        +------------+
|                | UART Comm.     |        | Fault      |
|                | Task           |        | Monitoring |
|                +-------+--------+        | Task       |
|                        |                 +------------+
+------------------------|------------------------------+
                         |
                       UART
                         |
                         v
                  Serial Terminal
