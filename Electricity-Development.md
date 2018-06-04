### Electricity Development notes and progress

#### The Idea:

To simulate a simple version of real world circuits. We are assuming that 1 wire actually equals a 2 pair wire and everything attached to the circuit is always wired in parallel. Also we need to avoid keeping lists of equipment, just like in the real world a circuit should calculate its own resistance and apply this load on any power sources on the point of connection. 

#### Direction of flow:
 
The StructureWired prefab and the IOElectricity interface accounts for a direction start and end indicator but electricity flow in circuits on unitystation run in all directions (because we assume everything is wired in parallel). This will simplify the circuits.

### The IOElectricty Interface:





