**State Diagram** for the **Elevator System**

```plaintext
                               +-------------------+
                               |       Idle        |
                               +-------------------+
                                       |
             Floor request received    |   Request for current floor
                                       v
                              +-------------------+
                              |   Moving Up/Down  |
                              +-------------------+
                                 |          |
     Arrive at floor requested   |          | No more requests
                                 v          |
                          +-------------------+ 
                          |    Door Opening   |
                          +-------------------+
                                 |
                      Doors fully open         v
                                 |         +----------------+
                                 |         |   Overloaded    |
                Passengers enter |         +----------------+
                     or exit     |                ^
                                 v                |
                          +-------------------+  |
                          | Loading/Unloading |  |
                          +-------------------+  |
                                 |                |
          Load below max weight  |                |  Weight exceeds max capacity
                                 v                |
                          +-------------------+  |
                          |    Door Closing   |  |
                          +-------------------+  |
                                 |                |
              Doors fully closed |                |
                                 v                |
                   Pending requests? -------------+
                         |   |
               Yes        |   | No
                         v    v
                  +--------------------+   
                  |   Moving Up/Down   |
                  +--------------------+   
                         | 
                 Arrive at next floor
                         v
                +---------------------+
                |   Door Opening      |
                +---------------------+
```

### **Detailed Breakdown of States and Transitions:**

#### **1. Idle**
   - **Description**: The elevator is not servicing any requests, and it's stationary at a floor.
   - **Transitions**:
     - **To Moving Up/Down**: When a new floor request is received from the controller or a passenger inside the elevator.
     - **To Door Opening**: If the request is for the current floor (no movement required).
  
#### **2. Moving Up/Down**
   - **Description**: The elevator is in transit between floors to service a request.
   - **Transitions**:
     - **To Door Opening**: When the elevator reaches the requested floor.
     - **To Idle**: If there are no further requests in the queue.

#### **3. Door Opening**
   - **Description**: The elevator arrives at a requested floor and starts opening its doors.
   - **Transitions**:
     - **To Loading/Unloading**: Once the doors are fully open, passengers start entering or exiting the elevator.
     - **To Overloaded**: If passengers enter and the weight exceeds the safe limit during loading.

#### **4. Loading/Unloading**
   - **Description**: Passengers are entering or exiting the elevator. The system monitors the load inside the elevator using weight sensors.
   - **Transitions**:
     - **To Overloaded**: If the total weight exceeds the capacity.
     - **To Door Closing**: If the load is below the maximum limit and all passengers have entered or exited.

#### **5. Door Closing**
   - **Description**: After passengers have entered or exited, the elevator's doors begin to close.
   - **Transitions**:
     - **To Moving Up/Down**: If there are more requests to handle, the elevator resumes movement to service the next request.
     - **To Idle**: If no more requests are pending, the elevator goes back to the `Idle` state.

#### **6. Overloaded**
   - **Description**: The elevator's weight sensors detect that the load exceeds the maximum allowed weight (e.g., 680 kg), and the elevator cannot move.
   - **Transitions**:
     - **To Loading/Unloading**: Once passengers exit and the weight falls below the threshold, the system returns to the loading state.
     - The elevator remains in this state until the load is reduced to safe levels.

#### **Edge Case Transitions:**
   - **Arrive at Floor but No Passengers Enter**: The elevator will proceed to `Door Closing` after a timeout if no passengers enter or exit after doors are fully opened.
   - **Emergency (Not Shown)**: In case of an emergency, the elevator might transition to an emergency handling state where it bypasses normal operations and returns to a designated floor (like the ground floor).

---

### **State Transitions Summary:**

1. **Idle** → **Moving Up/Down**:
   - Trigger: A floor request is received.
2. **Moving Up/Down** → **Door Opening**:
   - Trigger: The elevator reaches the requested floor.
3. **Door Opening** → **Loading/Unloading**:
   - Trigger: Doors fully open, and passengers enter/exit.
4. **Loading/Unloading** → **Overloaded**:
   - Trigger: Weight exceeds the maximum allowed.
5. **Overloaded** → **Loading/Unloading**:
   - Trigger: Passengers exit, reducing the load below the limit.
6. **Loading/Unloading** → **Door Closing**:
   - Trigger: Passengers finish entering/exiting, and load is below max weight.
7. **Door Closing** → **Moving Up/Down**:
   - Trigger: Pending requests in the queue.
8. **Door Closing** → **Idle**:
   - Trigger: No further requests to handle.

---

**Central Controller State Diagram**

```plaintext
                                +-------------------+
                                |       Idle        |
                                +-------------------+
                                       |
               Receive floor request   |  No requests
                                       v
                            +------------------------+
                            |   Processing Request   |
                            +------------------------+
                                       |
           Elevators available?        |   No elevators available
                Yes                    |   or system error
                |                      v
                v               +------------------------+
     +--------------------+     |     Error Handling     |
     | Dispatch elevator   |     +------------------------+
     | and update status   |             ^
     +--------------------+              |
                |                         |
                |                Error resolved or fallback
                v
      +-------------------------+
      |  Awaiting Elevator Move  |
      +-------------------------+
                |
     Elevator arrives at floor  | No further requests
                v               v
        +------------------+  +------------------+
        |    Update Queue   |  |       Idle       |
        +------------------+  +------------------+
                |
     More requests in queue?
             Yes | No
                v
      +-------------------------+
      |   Processing Request    |
      +-------------------------+
```

### **Detailed Breakdown of the Central Controller’s States and Transitions:**

#### **1. Idle**
   - **Description**: The Central Controller is in a waiting state, where no new requests have been received, and all elevators are either idle or handling existing requests.
   - **Transitions**:
     - **To Processing Request**: When a new floor request is received from either the External Request Panel or from inside an elevator.

#### **2. Processing Request**
   - **Description**: The Central Controller evaluates the request by checking the availability of all elevators and deciding which elevator is most suitable based on proximity, direction, load, and operational status.
   - **Transitions**:
     - **To Error Handling**: If no elevators are available or a system error occurs (e.g., all elevators are full or out of service).
     - **To Dispatch Elevator**: If one or more elevators are available, the system selects the best one and dispatches it to service the request.

#### **3. Dispatch Elevator**
   - **Description**: The Central Controller sends a command to the selected elevator, adding the floor request to the elevator’s queue. It also updates its own internal status to keep track of which elevator is handling the request.
   - **Transitions**:
     - **To Awaiting Elevator Move**: After dispatching, the system waits for the elevator to complete the request (move to the requested floor).

#### **4. Awaiting Elevator Move**
   - **Description**: The controller monitors the progress of the elevator after it has been dispatched. The system continuously receives position updates from the elevator.
   - **Transitions**:
     - **To Update Queue**: When the elevator reaches the requested floor, the system updates the elevator's queue (removes the request from its active queue).
     - **To Idle**: If the elevator reaches the requested floor and no further requests are pending in the system.
  
#### **5. Update Queue**
   - **Description**: The Central Controller updates its internal queue to remove the completed request from the elevator’s list of tasks. The system then checks if there are more pending requests.
   - **Transitions**:
     - **To Processing Request**: If there are more requests in the queue, the system returns to the request processing state to handle them.
     - **To Idle**: If no more requests are in the queue, the system transitions back to the Idle state.

#### **6. Error Handling**
   - **Description**: The Central Controller enters the error handling state when no elevators are available (e.g., all are busy, out of service, or overloaded) or if there’s a system fault.
   - **Transitions**:
     - **To Idle**: Once the error condition has been resolved or a fallback mechanism is triggered (e.g., another elevator becomes available).
     - **To Processing Request**: Once the system is ready to handle the pending request again, it returns to the request processing state.

---

### **Detailed State Transitions with Triggers and Events**:

1. **Idle** → **Processing Request**:
   - **Trigger**: A new floor request is received.
   
2. **Processing Request** → **Error Handling**:
   - **Trigger**: No elevators are available (e.g., all are full, out of service, or system error occurs).
   
3. **Processing Request** → **Dispatch Elevator**:
   - **Trigger**: One or more elevators are available, and the controller selects the most suitable one based on proximity, direction, and load.
   
4. **Dispatch Elevator** → **Awaiting Elevator Move**:
   - **Trigger**: The elevator is successfully dispatched and begins moving to the requested floor.

5. **Awaiting Elevator Move** → **Update Queue**:
   - **Trigger**: The elevator arrives at the requested floor, and the doors open.
   
6. **Update Queue** → **Processing Request**:
   - **Trigger**: More pending requests are in the system.
   
7. **Update Queue** → **Idle**:
   - **Trigger**: No more requests are pending in the system.
   
8. **Error Handling** → **Idle**:
   - **Trigger**: Error is resolved, and the system returns to normal operation.
   
9. **Error Handling** → **Processing Request**:
   - **Trigger**: The system is ready to handle the pending request again (e.g., a previously unavailable elevator becomes available).

---
