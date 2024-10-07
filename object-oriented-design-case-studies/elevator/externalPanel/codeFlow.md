# (Passenger Request via Outside Control Panel)
---

### **1. Passenger Presses the Up/Down Button**

When the passenger presses the Up/Down button, the **External Request Panel** sends a request to the **Central Controller**. Here's a simplified **TypeScript** implementation for handling this input.

#### **External Request Panel Code**:

```typescript
// ExternalRequestPanel.ts
class ExternalRequestPanel {
  constructor(private floor: number, private direction: 'UP' | 'DOWN') {}

  pressButton() {
    const request = {
      event_type: "FLOOR_REQUEST",
      floor: this.floor,
      direction: this.direction
    };

    // Send the request to the Central Controller
    CentralController.receiveFloorRequest(request);
  }
}
```

#### **Example Usage**:

```typescript
// Passenger presses the up button on floor 5
const panel = new ExternalRequestPanel(5, 'UP');
panel.pressButton();
```

---

### **2. Central Controller Receives and Processes the Request**

Once the Central Controller receives the request, it processes it by querying the status of all elevators to find the best one to handle the request.

#### **Central Controller Code**:

```typescript
// CentralController.ts
class CentralController {
  static elevators: Elevator[] = [new Elevator(1), new Elevator(2), new Elevator(3)];

  // Handles floor requests from external panels
  static receiveFloorRequest(request: { event_type: string, floor: number, direction: 'UP' | 'DOWN' }) {
    console.log(`Received request for floor ${request.floor} in direction ${request.direction}`);
    this.dispatchElevator(request);
  }

  // Dispatches the best elevator based on the request
  static dispatchElevator(request: { floor: number, direction: 'UP' | 'DOWN' }) {
    let bestElevator: Elevator | null = null;
    let bestScore = Infinity;

    this.elevators.forEach(elevator => {
      const score = elevator.calculateScore(request.floor, request.direction);
      if (score < bestScore && elevator.canHandleRequest(request)) {
        bestScore = score;
        bestElevator = elevator;
      }
    });

    if (bestElevator) {
      console.log(`Dispatching elevator ${bestElevator.id} to floor ${request.floor}`);
      bestElevator.addRequestToQueue(request);
    } else {
      console.log("No elevators available.");
    }
  }
}
```

#### **Elevator Class**:

The elevator calculates its score (how suitable it is to handle the request) and decides whether it can handle the new request.

```typescript
// Elevator.ts
class Elevator {
  currentFloor: number = 0;
  direction: 'UP' | 'DOWN' | 'IDLE' = 'IDLE';
  load: number = 0;  // in kilograms
  requestQueue: { floor: number, direction: 'UP' | 'DOWN' }[] = [];

  constructor(public id: number) {}

  // Calculate the proximity score to decide if this elevator is suitable for the request
  calculateScore(floor: number, direction: 'UP' | 'DOWN'): number {
    // If the elevator is idle, it's a good candidate
    if (this.direction === 'IDLE') return Math.abs(this.currentFloor - floor);

    // If the elevator is moving in the requested direction, prioritize it
    if (this.direction === direction) return Math.abs(this.currentFloor - floor);

    // Otherwise, deprioritize this elevator
    return Infinity;
  }

  // Check if the elevator can handle the request (e.g., it's not overloaded)
  canHandleRequest(request: { floor: number, direction: 'UP' | 'DOWN' }): boolean {
    return this.load < 680;  // Assume max load is 680 kg
  }

  // Add a new floor request to the elevator's queue
  addRequestToQueue(request: { floor: number, direction: 'UP' | 'DOWN' }) {
    this.requestQueue.push(request);
    this.processNextRequest();
  }

  // Process the next request in the queue
  processNextRequest() {
    if (this.requestQueue.length > 0) {
      const nextRequest = this.requestQueue[0];
      this.moveToFloor(nextRequest.floor);
    }
  }

  // Simulate moving the elevator to the requested floor
  moveToFloor(targetFloor: number) {
    console.log(`Elevator ${this.id} moving from floor ${this.currentFloor} to ${targetFloor}`);
    this.currentFloor = targetFloor;  // Simulate the move
    this.openDoors();
  }

  // Simulate opening the doors
  openDoors() {
    console.log(`Elevator ${this.id} doors are open on floor ${this.currentFloor}`);
    this.requestQueue.shift();  // Remove the processed request
    this.processNextRequest();  // Continue with the next request
  }
}
```

#### **Example Interaction**:

```typescript
// Simulating a request from floor 5, up direction
const panel = new ExternalRequestPanel(5, 'UP');
panel.pressButton();
```

Output:
```
Received request for floor 5 in direction UP
Dispatching elevator 1 to floor 5
Elevator 1 moving from floor 0 to 5
Elevator 1 doors are open on floor 5
```

---

### **3. Real-Time Status Updates and Elevator Movement**

The elevator regularly updates the Central Controller about its current status (e.g., position, load, etc.) to ensure the system can make real-time decisions.

#### **Elevator Sends Status Updates**:

```typescript
class Elevator {
  // Other methods...

  sendStatusUpdate() {
    const status = {
      elevator_id: this.id,
      current_floor: this.currentFloor,
      direction: this.direction,
      load: this.load
    };
    CentralController.receiveStatusUpdate(status);
  }
}
```

#### **Central Controller Receives Status Updates**:

```typescript
class CentralController {
  // Other methods...

  static receiveStatusUpdate(status: { elevator_id: number, current_floor: number, direction: string, load: number }) {
    console.log(`Elevator ${status.elevator_id} is on floor ${status.current_floor}, moving ${status.direction} with load ${status.load}kg`);
  }
}
```

#### **Example Elevator Status Update**:

```typescript
// Simulate elevator status updates
elevator.sendStatusUpdate();
```

Output:
```
Elevator 1 is on floor 5, moving UP with load 400kg
```

---

### **4. Handling Overload**

When an elevator exceeds its weight capacity, it should prevent further movement and notify the passengers and system.

#### **Overload Handling Code**:

```typescript
class Elevator {
  // Other methods...

  checkOverload() {
    if (this.load > 680) {
      console.log(`Elevator ${this.id} is overloaded! Please reduce load.`);
      this.openDoors();
    }
  }
}
```

#### **Simulating an Overload**:

```typescript
// Simulating an elevator overload
elevator.load = 700;  // Exceeds the 680 kg limit
elevator.checkOverload();
```

Output:
```
Elevator 1 is overloaded! Please reduce load.
Elevator 1 doors are open on floor 5
```

---

### **5. Feedback to the External Display Panel**

The Central Controller sends feedback to the **External Display Panel** to show which elevator has been dispatched and when it will arrive.

#### **Code for Updating External Display Panel**:

```typescript
class ExternalDisplayPanel {
  static updateDisplay(message: string) {
    console.log(`External display: ${message}`);
  }
}

class CentralController {
  // After dispatching an elevator
  static dispatchElevator(request: { floor: number, direction: 'UP' | 'DOWN' }) {
    let bestElevator = null;
    let bestScore = Infinity;

    // Find the best elevator...
    // Dispatching logic...

    if (bestElevator) {
      bestElevator.addRequestToQueue(request);
      ExternalDisplayPanel.updateDisplay(`Elevator ${bestElevator.id} is on the way`);
    }
  }
}
```

#### **Example External Display Update**:

```typescript
// Simulate sending feedback to the external panel
ExternalDisplayPanel.updateDisplay("Elevator 2 is on the way");
```

Output:
```
External display: Elevator 2 is on the way
```

---

### **End-to-End Flow Example**

Now, putting it all together, here's how a typical flow works in sequence:

1. Passenger presses the Up button on floor 5.
2. The Central Controller receives the request and evaluates all elevators.
3. The most suitable elevator (based on direction, proximity, and load) is selected.
4. The elevator is dispatched to floor 5, and the external display is updated to notify the passenger.
5. The elevator moves to floor 5, opens its doors, and allows passengers to enter.
6. The elevator updates its internal queue and processes the next request.
