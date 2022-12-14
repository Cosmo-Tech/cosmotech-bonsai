# Inventory Management sample
Demo instance of the Cosmo Tech Inventory Management conceptual model

## Use case
Inventory management of supply chain inventories under uncertain and volatile demand.

## Overview
The model represents a simple supply chain that consists of two retailers, a single supplier and a centralized storage hub. Each one of these facilities has a single inventory. The demand received by the retailers is uncertain, fluctuating and strong unpredictable peaks can occur. Inventories at each retailer must decide how much to order each week, what percentage of these orders must be sent directly to the supplier to be received quickly by plane, and what percentage can be ordered to the central hub and received by truck. The central hub send orders to the supplier, it must decide how much to order each week to be able to respond to the retailers' orders but taking into account costs of storage. It can store large quantities and is located near the retailers so that shipments can be delivered quickly by truck. Shippments from the supplier to the hub are sent by train, the travel time is longer and uncertain. Plane shipping is much more expensive but it is received faster to avoid backlog penalties.

Uncertainties in demand and transport time, trade-offs between delivery speed and transport and storage costs, competition between retailers at each time step for available upstream stocks, lags between decisions and the results of these decisions, and coupling and interferences between individual decisions, strongly hinders the ability of a supply chain manager to take optimal decisions.

An intelligent global control policy for the management of the supply chain inventories is thus needed. It must decide how much each retailer must order directly to the supplier and how much to the hub, and how much the hub must order, at each time step, in order to be able to respond to the demand, reduce costs and maximize profit. In addition, it must respect eventual stocks constraints and minimum service level requirements, and all this under an uncertain and volatile demand.

### Inventory Management conceptual model

The inventory management simulator provided for this sample use case is an instance of an Inventory Management model. Other more complex instances of this model can easily be built for different and more complex scenarios and use cases (see https://cosmotech.com/platform/ for more information). The following is a short summary of this conceptual model, where only the most important elements, useful for a better understanding of the provided instance simulator and is interaction with the Bonsai brain, are mentioned.

### Main entity types: 
There are three main entity types : SupplyChain, Inventory and Facility. Facility is an abstract type and is thus not used explicitly in the sample instance model; instead, there are three concrete types of Facility, two of them used in the sample model (Retailer, Supplier), a third, ProductionOperation, is not used in this instance but is available for more complex models.

#### Hierarchical structure and control
A SupplyChain is composed of Facilities that contain Inventories, as well as Information and Logistics Networks that provide Inventories with communication and transport structures (see section below). Retailers entities are responsible for the reception and management of demands, the service of demands with existing on-hand quantities in its Inventories, and the computation of local demand and service indicators. Suppliers are responsible for the production of quantities according to their own current capacity and to incomming orders received at their inventories. Inventories themselves are responsible for the local management of orders, they collect quantities received from upstream inventories, prepare shipments to be sent to downstream inventories, and set the quantites to be ordered and their source allocations according to available predefined policies or to externally provided values (such as Bonsai's brain actions).   

#### Interaction networks, communication and transport
All communication and transport interactions occur at the inventory level, quantities are shipped and orders are sent from one inventory to another through the information and logistics networks, respectively. 
Orders are transmitted without delay through communication channels from one inventory to another. Shipments, on the other hand, can take some time to be delivered and this delay can be uncertain; Transport type entities are in charge of this process.

### Instance model

Quantities travel throuhg two different kind of transports

### Default simulator initial configuration

The following parameter initial values are given for information about the simulator behavior. Note however that these variables are not all available for configuration from the inkling file (for those values, refer to 'Connection to Bonsai -> Simulator configuration' below).

* Supplier:
 * Capacity: infinite


### Processes scheduling and global Brain-Simulator behavior
The global behavior of the simulator and the impact that a Bonsai's brain actions have on it during training depend both on the ordering of processes execution within the model and on the place of Bonsai's brain actions within that ordering. The following sequence summarizes this processes scheduling:

* For each Retailer: set demand on own inventory
* Repeat N times:
  * Bonsai brain: reads state
  * Bonsai brain: send actions (set quantities to order and source allocations on Retailer1, Retailer2 and Hub Inventories)
  * For each inventory (1,2,Hub) : set quantities to order and source allocation to those set by the brain, send orders to upstream inventories
  * Supplier: produce quantities
  * Supplier inventory: collect production and prepare shipments
     * For each Transport from supplier (Plane1, Plane2,Train): collect shipments, advance shipments being transported, prepare and deliver shipments
  * Hub inventory: collect input quantities and prepare shipments
  * For each Transport from Hub (Truck1, Truck2):  collect shipments, advance shipments being transported, prepare and deliver shipments arriving at destination
  * For each Retailer:
    * Own inventory: collect received quantities and prepare them for Retailer
    * Serve demand
  * Update persistent state variables: HoldingCost (Inventories), Supplier Capacity,
  * Clear temporal state variables: 
  * Increment time
  * For each Retailer: 
    * Update CurrentDemand and CurrentDemandWithBacklog
    * Set Inventory::Demand with CurrentDemand
  
Note that most of the values of states available for teaching or monitoring in Bonsai are those at the end of each Repeat block, whereas the CurrentDemand value is the new value for the new simulation step loaded at the end of the Repeat block.  That is, the value of the CurrentDemand (and CurrentDemandWithBacklog) is in advance of all other states values by one simulation step. Therefore, values for other states such as the FulfilledDemand correspond to the previous step CurrentDemand.

### Connection to Bonsai

#### Available simulator states
The following state variables are made available to Bonsai at each simulation step through the connector interface.
From these variables, the IncrementProfit, CurrentServiceLevel, OnHandInventory(\_1,\_2,\_Hub) and AlreadyOrderedQuantity(\_1,\_2,\_Hub) have been made available for training in the inkling file, all other variables are made available for visualization of the simulator state during training or assessment.

Note that it is possible to change the inkling file to use other set of variables for training if you want to test an alternative training.

##### For training :
On hand quantity and already ordered quantities for each Inventory.
* IncrementProfit (float): Increment of aggregated profit over both retailers at each simulation step (float)
* CurrentServiceLevel (float in [0,1]) : Average of ImmediatelyFulfilledDemand(at current step)/CurrentDemand(at previous step) for both retailers 
* OnHandInventory_1, OnHandInventory_2, OnHandInventory_Hub: current stock at each (non-supplier) inventory (all float)
* AlreadyOrderedQuantity_1, AlreadyOrderedQuantity_2, AlreadyOrderedQuantity_Hub : cummulated quantity already ordered by each (non-supplier) inventory (all float)
* CurrentDemand_1, CurrentDemand_2: Demand for each retailer at the current time step (both float)

##### Additional simulator outputs used for monitoring, not used for training
* BacklogQuantity_1, BacklogQuantity_2: cummulated unserved demand for each retailer (both float)
* QuantityOrderedByPlane_1,QuantityOrderedByPlane_2, QuantityOrderedThroughHub_1, QuantityOrderedThroughHub_2: (all float),
* QuantityAtDestination_Truck1, QuantityAtDestination_Truck2, QuantityAtDestination_Plane1, QuantityAtDestination_Plane2, QuantityAtDestination_Train: (all float)
* FulfilledDemand_1, FulfilledDemand_2: cummulated demand that has been served (either immediatly or not, up to the previous step) by each Retailer (both float)
* ImmediatlyFulfilledDemand_1, ImmediatlyFulfilledDemand_2: demand that has been served immediatly (at the previous step) by each Retailer (both float)

#### Brain actions
The bonsai brain actions override and control the inventories' decision policies through the simulation controler. There are two different decisions produced by the brain for each inventory: the quantity to be ordered and the allocation of that order to upstream inventories. These actions are, for the sample instance:
* Order_Inventory1, Order_Inventory2 and Order_InventoryHub (Quantities ordered by each retailer and by the hub, positive real numbers)
* Inventory1_AllocateToHub and Inventory2_AllocateToHub (proportion ordered by each Retailer that is allocated to the Hub, real number between 0 and 1, the complement is ordered directly to the supplier)

#### Simulator configuration
The following simulator parameters are available for configuration from the inkling file. Note that you can change the inkling file to set different values for this parameters if you want to test alternative trainings.  
    
* HoldingCostPerPiece_Inventory1, HoldingCostPerPiece_Inventory2, HoldingCostPerPiece_InventoryHub: holding costs per piece for Retailer 1, Retailer 2 and Hub inventories(all float; default values are: 2, 2 and 3 respectively)
* TransportCostPerPiece_Plane1, TransportCostPerPiece_Plane2, TransportCostPerPiece_Truck1, TransportCostPerPiece_Truck2, TransportCostPerPiece_Train: transport costs per piece for each Transport entity (all float; default values are 25,25,3,3 and 5, respectively)



