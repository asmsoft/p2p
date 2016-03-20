Реализовать приложение, работающее в одноранговой P2P сети. 
===========================================================

Базовый функционал:
-------------------
1. Любой узел может принять на вход сообщение.
2. Узел должен разослать сообщение всем остальным узлам в сети.
3. Каждый узел хранит у себя историю сообщений.
4. Необходим алгоритм консенсуса (новый узел при включении в сеть должен синхронизироваться автоматически; при сбое в работе узла он должен понять, что произошел сбой и синхронизироваться с сетью)

Дополнительные сведения:
------------------------
Формат сериализации сообщений и протокол общения между узлами - на свой выбор, с обоснованием, почему выбраны конкретные решения. В рамках тестовой задачи можно считать, что узлы находятся в одном VLAN


Transaction flow:
-----------------

```
Client               Node                         Network
-------              -----                        -------
   |                   |                             |
   | Incoming message  |                             |
   |------------------>|                             |
   | Message accepted  |                             |
   |<------------------|                             |
   |                   |                             |   
   |                   | Start Transaction           |   
   |                   |---------------------------->|
   |                   |                             |
   |                   | Confirm Transaction Started |
   |                   |<----------------------------|
   |                   |                             |   
   |                   | Message Packet              |
   |                   |---------------------------->|
   |                   |                             |   
   |                   | Message Received            |
   |                   |<----------------------------|
   |                   |                             |   
   |                   | Commit                      |
   |                   |---------------------------->|
   
   
      
```

Synchronization flow:
---------------------

```
Node                         Network          Selected Node
-----                        -------          -------------
  |                             |                   |
  | Syncronization Request      |                   |
  |---------------------------->|                   |
  |                             |                   |
  | Versions responses          |                   |
  |<============================|                   |
  |                             |                   |
  | Select a node having max v  |                   |
  |---------+                   |                   |
  |         |                   |                   |
  |<--------+                   |                   |
  |                             |                   |
  | Get last DB request         |                   |
  |------------------------------------------------>|
  |                             |                   |
  | Last DB dat                 |                   |
  |<------------------------------------------------|
  |                             |                   |
  | Update local cache          |                   |
  |--------+                    |                   |
  |        |                    |                   |
  |<-------+                    |                   |
  |                             |                   |
  
```

Heartbeat:
----------

```
Node                         Network
-----                        -------
  |                             |
  | Ping broadcast (T=100ms)    |
  |---------------------------->|
  |                             |
    
Node                         Network
-----                        -------
  |                             |
  | Incoming Ping broadcast     |
  |<----------------------------|
  |                             |
  | Refresh nodelist            |
  |--------+                    |
  |        |                    |
  |<-------+                    |
  |                             |
  
```

Node state machine:
-------------------

```
+---------------+  Update   +-----------+
|               |---------->|           |
| InTransaction |           |  Updated  |
|               |           |           |
+---------------+           +-----------+
    ^                            | |          
    |                            | | 
    | Start transaction          | | Commit/Rollback 
    | received                   | |
    |                            V V
    |                     +--------------+       Synchronize      +------------------+
    +---------------------|              |----------------------->|                  |
                          |  Connected   |       Synchronized     |  Synchronization |
    +---------------------|              |<-----------------------|                  |
    |                     +--------------+                        +------------------+
    |                          ^   ^
    | Incoming message         |   |                    Yield
    | received                 |   +--------------------------------------------------------------+
    |                          |       Commit                                                     |
    |                          +---------------------------------------+                          |
    V                                                                  |                          |
+-------------------+  Start       +--------------------+  Update  +----------+  Rollback  +---------------------+
|                   |------------->|                    | -------->|          |----------->|                     |
|  IncomingReceived |  transaction | StartedTransaction |          | Updating |            |  TransactionFailed  |
|                   |              |                    |          |          |            |                     |
+-------------------+              +--------------------+          +----------+            +---------------------+
  
```
