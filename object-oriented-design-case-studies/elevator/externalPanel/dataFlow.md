# (Passenger Request via Outside Control Panel)
---

### **1. Passenger Presses the Up/Down Button**

#### **Input from External Control Panel**:
When a passenger presses the **Up** or **Down** button, the control panel generates a request. The data sent to the Central Controller looks like:

```json
{
  "event_type": "FLOOR_REQUEST",
  "floor": 5,
  "direction": "UP"
}
```

- **event_type**: Identifies the type of request (a floor request in this case).
- **floor**: Specifies which floor the request was made from.
- **direction**: Indicates whether the passenger wants to go up or down.

#### **Central Controller Receives and Processes**:
The Central Controller receives the data, logs it, and checks the status of all elevators to decide which elevator should service the request.

### **2. Central Controller Queries Elevator Status**

#### **Data Structure for Elevator Status Request**:
The Central Controller queries the status of all elevators in real time to find the best one to assign to the floor request. Here’s the typical structure of a **status request** sent to each elevator:

```json
{
  "request_type": "ELEVATOR_STATUS_QUERY",
  "elevator_id": 1
}
```

- **request_type**: Specifies the type of request (a status query in this case).
- **elevator_id**: Identifies the specific elevator being queried.

#### **Response from Each Elevator**:
Each elevator responds to the status query with its current state. Here's an example of a response:

```json
{
  "elevator_id": 1,
  "current_floor": 7,
  "direction": "IDLE",
  "load": 450,  // in kilograms
  "max_load": 680,  // in kilograms
  "request_queue": [
    { "floor": 10, "direction": "UP" }
  ],
  "operational_status": "OPERATIONAL"
}
```

- **elevator_id**: Identifies the responding elevator.
- **current_floor**: Indicates the current floor where the elevator is located.
- **direction**: Shows whether the elevator is moving up, down, or idle.
- **load**: The current load (e.g., weight) inside the elevator.
- **max_load**: The maximum load capacity of the elevator.
- **request_queue**: A list of the elevator’s current requests (e.g., it’s going to floor 10 in the upward direction).
- **operational_status**: Whether the elevator is operational or out of service.

#### **Additional Considerations for the Controller**:
- If an elevator is **out of service** or **at max load**, it won’t be considered.
- The **proximity** and **direction** relative to the request will be used to make a decision on the best elevator.

### **3. Dispatch Algorithm Decision** (Elevator Selection)

Once the Central Controller collects data from all elevators, it uses a dispatch algorithm to determine the best elevator for the request.

#### **Sample Decision**:
The following is an internal representation of how the decision might be made:

```json
{
  "elevator_selected": 2,
  "reason": "Nearest elevator moving in requested direction",
  "elevators_considered": [
    { "elevator_id": 1, "score": 8 },
    { "elevator_id": 2, "score": 3 },  // Best score (closest in direction)
    { "elevator_id": 3, "score": 10 }
  ]
}
```

- **elevator_selected**: The ID of the elevator that has been selected to service the request.
- **reason**: The logic behind the selection, explaining why this particular elevator was chosen.
- **elevators_considered**: A list of all elevators evaluated, along with a **score** representing how optimal they are (lower score = better match). The score is based on factors like proximity, direction, and load.

### **4. Update the Elevator’s Request Queue**

#### **Command to the Selected Elevator**:
Once an elevator is selected, the Central Controller updates its request queue. Here’s an example of the payload sent to the selected elevator (Elevator 2 in this case):

```json
{
  "command_type": "UPDATE_QUEUE",
  "elevator_id": 2,
  "new_request": {
    "floor": 5,
    "direction": "UP"
  }
}
```

- **command_type**: Specifies the type of command being sent (queue update).
- **elevator_id**: Identifies the elevator to which the command is being sent.
- **new_request**: The new floor request being added to the elevator’s queue.

#### **Elevator’s Internal Queue After Update**:
Once the elevator updates its internal queue, it will look something like this:

```json
{
  "elevator_id": 2,
  "current_floor": 3,
  "request_queue": [
    { "floor": 5, "direction": "UP" },
    { "floor": 12, "direction": "UP" }
  ]
}
```

- The new request (floor 5, up) is added to the elevator’s queue.
- The elevator will stop at floor 5 next before moving on to service the rest of its queue.

### **5. Feedback to External Request Panel**

#### **Update to External Display Panel**:
After the elevator is assigned, the Central Controller sends feedback to the external panel on the floor where the request was made. The payload might look like this:

```json
{
  "display_update": {
    "message": "Elevator 2 is on the way",
    "current_elevator_position": 3
  }
}
```

- **message**: Tells the passenger which elevator has been assigned to their request.
- **current_elevator_position**: Shows the current floor of the assigned elevator so the passenger knows how far away it is.

### **6. Elevator Movement and Status Updates**

#### **Position Updates from the Elevator**:
As the elevator moves toward the requested floor, it continuously sends position updates to the Central Controller. Example data sent from the elevator might look like:

```json
{
  "elevator_id": 2,
  "current_floor": 4,
  "direction": "UP",
  "estimated_arrival_time": 5  // in seconds
}
```

- **current_floor**: Updates the current position of the elevator.
- **direction**: Confirms that the elevator is still moving in the correct direction.
- **estimated_arrival_time**: Provides an estimated time of arrival based on the current speed and distance.

#### **System Updates for Arrival**:
When the elevator arrives at the requested floor, it will send a final arrival notification:

```json
{
  "elevator_id": 2,
  "current_floor": 5,
  "status": "ARRIVED",
  "doors_open": true
}
```

- **status**: Indicates that the elevator has arrived at the requested floor.
- **doors_open**: Confirms that the doors have been opened for the passenger.

### **7. Load Handling and Capacity Checks**

#### **Weight Sensor Data**:
When passengers enter the elevator, the weight sensors continuously monitor the load to ensure it does not exceed capacity. Sample data from the weight sensor might look like:

```json
{
  "elevator_id": 2,
  "current_load": 500,  // in kilograms
  "max_load": 680,      // in kilograms
  "status": "OK"
}
```

- **current_load**: The current weight of passengers inside the elevator.
- **max_load**: The maximum allowed weight for safe operation.
- **status**: Indicates whether the load is within safe limits (in this case, "OK").

#### **Overload Scenario**:
If the elevator becomes overloaded, it sends an alert:

```json
{
  "elevator_id": 2,
  "current_load": 700,
  "status": "OVERLOAD",
  "doors_open": true
}
```

- **status**: The elevator enters an "OVERLOAD" state, preventing it from moving.
- **doors_open**: The doors remain open to allow passengers to exit until the load is reduced.

---

### **Key Points in Data Flow**
- **Request Payloads**: When a floor request is made, the system handles the request by transmitting structured data to the Central Controller.
- **Real-Time Status Updates**: Elevators constantly communicate their position, direction, load, and operational status to the Central Controller.
- **Queue Management**: The controller updates each elevator’s queue based on dynamic requests, ensuring efficient dispatching and scheduling.
- **Feedback to Users**: Throughout the process, the system sends real-time feedback to external panels, giving passengers clear information about which elevator is coming and when it will arrive.

---
