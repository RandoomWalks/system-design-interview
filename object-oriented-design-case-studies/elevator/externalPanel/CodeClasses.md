
---

### **1. Class: CentralController**
The **CentralController** is responsible for managing all elevators, processing requests, and making dispatch decisions. It coordinates the system and ensures efficient handling of floor requests.

#### **Attributes**:
- `elevators: Elevator[]`: Array holding all the elevators managed by the controller.
- `requests: Request[]`: Queue of active requests waiting to be fulfilled.
- `dispatchAlgorithm: DispatchAlgorithm`: Reference to the dispatch algorithm for choosing the optimal elevator.
- `floorPanels: ExternalRequestPanel[]`: References to all floor panels (input sources).
  
#### **Methods**:
- `receiveFloorRequest(request: Request): void`: Receives a request from an external panel and queues it for processing.
  - **Error Handling**: Checks if the request is valid (e.g., ensures the floor and direction are in the valid range).
  
- `dispatchElevator(request: Request): void`: Dispatches an elevator to fulfill the request by using the dispatch algorithm to determine the most suitable elevator.
  - **Preconditions**: All elevators should be operational unless flagged as out of service.
  - **Postconditions**: Once an elevator is assigned, the system updates the external panel to indicate which elevator is coming.
  - **Error Handling**: If no elevators are available, the request is placed in a pending state until one becomes available.

- `receiveStatusUpdate(status: ElevatorStatus): void`: Updates the controller's knowledge of the current state of each elevator (e.g., position, direction, load).
  - **Concurrency Handling**: Handles simultaneous status updates from multiple elevators.
  - **Timeout Handling**: If an elevator stops sending status updates (e.g., communication loss), it is flagged as out of service, and pending requests are redistributed.

#### **Relationships**:
- **Aggregation**: The `CentralController` aggregates multiple `Elevator` instances, managing their state and controlling their movements.
- **Association**: Uses `DispatchAlgorithm` to assign requests to elevators efficiently.

---

### **2. Class: Elevator**
The **Elevator** class models individual elevators in the system, managing movement, requests, and safety (e.g., overload detection). Each elevator processes requests assigned to it by the **CentralController**.

#### **Attributes**:
- `id: number`: Unique identifier for the elevator.
- `currentFloor: number`: The elevator’s current floor position.
- `direction: 'UP' | 'DOWN' | 'IDLE'`: Current movement state.
- `load: number`: Current weight inside the elevator (in kilograms).
- `maxLoad: number = 680`: Maximum allowable weight.
- `requestQueue: Request[]`: Queue of floor requests the elevator is responsible for.
- `isOperational: boolean = true`: Status flag indicating whether the elevator is operational.

#### **Methods**:
- `addRequestToQueue(request: Request): void`: Adds a floor request to the elevator’s queue and reorders it to optimize travel.
  - **Error Handling**: Prevents requests from being added if the elevator is overloaded or out of service.
  
- `moveToFloor(floor: number): void`: Moves the elevator to the requested floor.
  - **Safety Handling**: Checks if the path is clear (doors closed, no obstruction) and ensures the elevator doesn’t exceed speed limits.
  
- `openDoors(): void`: Opens the elevator doors when it reaches the target floor.
  - **Error Handling**: Ensures doors are only opened if the elevator has completely stopped and is aligned with the floor.
  
- `checkOverload(): boolean`: Checks whether the elevator is overloaded and prevents further movement if necessary.
  - **Error Handling**: Sends an overload notification to the controller if the load exceeds the maximum capacity.
  
- `calculateScore(request: Request): number`: Calculates a "suitability score" for handling the request based on proximity, direction, and load.
  - **Logic**: Lower score means higher priority for handling the request (e.g., proximity and direction matching heavily influence the score).

#### **Relationships**:
- **Composition**: Each `Elevator` contains internal sensors (like `WeightSensor` and `PositionSensor`), which monitor its state and provide feedback to the system.

---

### **3. Class: Request**
The **Request** class models the data structure for floor requests made by passengers either through external panels or inside the elevator.

#### **Attributes**:
- `floor: number`: The floor from which the request is made or the destination floor.
- `direction: 'UP' | 'DOWN'`: Specifies the direction of the request (Up for ascending, Down for descending).
- `isPriority: boolean = false`: Indicates whether the request is a high-priority request (e.g., emergency stop or service request).

#### **Methods**:
- `validate(): boolean`: Validates the request to ensure the floor number is within bounds, and the direction is logical (e.g., you can't request to go up from the top floor).
  - **Error Handling**: Throws exceptions or flags invalid requests.

#### **Relationships**:
- **Association**: Requests are created by passengers via `ExternalRequestPanel` or from the `Elevator`'s internal control panel.

---

### **4. Class: DispatchAlgorithm**
The **DispatchAlgorithm** class encapsulates the logic for selecting the best elevator to handle a floor request. This logic can range from simple proximity-based dispatching to more complex load-balancing algorithms.

#### **Methods**:
- `calculateBestElevator(request: Request, elevators: Elevator[]): Elevator | null`: Evaluates the state of all elevators and returns the most suitable elevator for the request.
  - **Error Handling**: If no elevator is found to handle the request (e.g., all elevators are at full capacity), the method returns `null`, and the controller places the request in a waiting queue.
  - **Algorithm Considerations**:
    - **Proximity**: Elevators closest to the requested floor are preferred.
    - **Direction**: Elevators already moving in the requested direction are preferred.
    - **Load**: Elevators with lighter loads are prioritized to avoid overload scenarios.

#### **Relationships**:
- **Association**: Used by `CentralController` to handle request dispatching logic.

---

### **5. Class: ExternalRequestPanel**
The **ExternalRequestPanel** represents the floor panel that passengers use to request an elevator. It forwards requests to the **CentralController** and provides feedback (e.g., which elevator is coming).

#### **Attributes**:
- `floor: number`: The floor where the panel is located.
- `direction: 'UP' | 'DOWN'`: The direction requested by the passenger.
  
#### **Methods**:
- `pressButton(direction: 'UP' | 'DOWN'): void`: Sends the floor request to the **CentralController**.
  - **Error Handling**: Prevents button presses if the elevator system is temporarily disabled (e.g., during maintenance).

#### **Relationships**:
- **Association**: Communicates with the `CentralController` to send requests.

---

### **6. Class: ElevatorStatus**
The **ElevatorStatus** class encapsulates status information sent from the elevator to the controller.

#### **Attributes**:
- `elevatorId: number`: The ID of the elevator sending the update.
- `currentFloor: number`: The elevator's current floor.
- `direction: 'UP' | 'DOWN' | 'IDLE'`: The elevator's current direction.
- `load: number`: Current load (weight in kilograms).
- `isOperational: boolean`: Status flag indicating whether the elevator is functioning normally.

#### **Methods**:
- `serialize(): string`: Converts the status object to a string (or JSON) for transmission over the network.
  
#### **Relationships**:
- **Association**: Status data is generated by `Elevator` instances and transmitted to the `CentralController` for monitoring and decision-making.

---

### **Class Diagram (Text-Based Overview)**

```plaintext
+----------------------+       +----------------------+
|   CentralController  |       |        Elevator       |
+----------------------+       +----------------------+
| - elevators: Elevator[]       | - id: number         |
| - requests: Request[]         | - currentFloor: int  |
| - dispatchAlgorithm: Dispatch | - direction: enum    |
+----------------------+       | - load: number        |
| + receiveFloorRequest()       | - maxLoad: number    |
| + dispatchElevator()          | - requestQueue: Req[]|
| + receiveStatusUpdate()       +----------------------+
+----------------------+       | + addRequestToQueue() |
                                | + moveToFloor()      |
          ^                     | + openDoors()        |
          |                     | + checkOverload()    |
          |                     +----------------------+
          |                            ^
          v                            |
+-------------------+                  |
|   Request         |                  |
+-------------------+                  |
| - floor: number   |                  |
| - direction: enum |                  |
| - isPriority: bool|------------------+
+-------------------+

+----------------------+       +----------------------+
|  DispatchAlgorithm   |       |  ExternalRequestPanel |
+----------------------+       +----------------------+
| + calculateBestElevator()    | - floor: number       |
|                              | - direction: enum     |
+----------------------+       +----------------------+
                               | + pressButton()       |
                               +----------------------+

+-----------------------+      +---------------------+
|    ElevatorStatus      |      |       Sensors

       |
+-----------------------+      +---------------------+
| - elevatorId: number   |      | Weight, Position    |
| - currentFloor: number |      +---------------------+
| - direction: enum      |
| - load: number         |
| - isOperational: bool  |
+-----------------------+
```

---
