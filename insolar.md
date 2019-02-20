# 1 func (lr *LogicRunner) CheckOurRole(ctx context.Context, msg core.Message, role core.DynamicRole) error

	*CheckOurRole* method performes authorization, checking the role retrieved from the Context against DynamicRole, 
	and returns error if authorization failed or nill value if not. Method has a receiver, that is a pointer to 
	LogicRunner type.

	Parameters:
		- ctx type: context.Context
		  Context type object of context package from where the currrent role data is retrieved.

		- msg type: core.Message
		  Message type object of core package used to record authorization data to default target 
		  (see [DefaultTarget] description).

		- role type: core.DynamicRole
		  DynamicRole (see [DynamicRole] description) object of core package to get the role data, which the current role 
		  will be checked against.

	Successful authorization
		return: nil
	Unsuccessful authorization
		return:
			- error type: string 
			  Error message, one of:
				* "authorization failed with error <error>" returns if an error occured, while authorization, 
				where <error> is a specific error number;
				* "can't execute this object" returns in case of unsuccessful role authorization.

func (lr *LogicRunner) executeMethodCall(ctx context.Context, es *ExecutionState, m *message.CallMethod) (core.Reply, error)
	Executes the contract when some CallMethod initiates contract execution. 
	When execution is successfuly finished *executeMethodCall* writes execution result to a *Result* field 
	and *current.Request* data to a *Request* field of the *reply.CallMethod* and returns a pointer to the 
	*reply.CallMethod*. *executeMethodCall* also deactivates contract object after executing the contract 
	and registers results of operation with ArtifactManager. Method has a receiver, that is a pointer to LogicRunner type.

	Parameters:
		- ctx type: context.Context
		  Context type object of context package from where the currrent context data (e.g. current contract descriptors) 
		  is retrieved.

		- es type: *ExecutionState
		  ExecutionState pointer for current contract object execution state.
		  
		- m type: *message.CallMethod
		  Pointer to the message from initiating call method. 

	Successful method execution:
		return: 
			- reply.CallMethod type:    
			CallMethod *reply* propertie with *result* and *request* fields, where *result* is a result 
			of CallMethod execution, performed by LogicRunner executor and *request* is the pointer to 
			*current.Request*.
			- nil type: error 
			  Nil error is returned for successful method execution.	
	
	Unsuccessful method execution:
		return:
			- nil 
			  Nil value is returned as a *core.Reply* for unsuccessful method results, details in *error* return value. 
			
			- error type: error 
			  Appropriate error code with error message, one of:
				* "couldn't get descriptors by object reference" in case of LogicRunner cant 
					get object descriptors from CallMethod message; 
		
				* "proxy call error: try to call method of prototype as method of another prototype" 
				  in case of method prototype cant be found in contract code;

				* "no executor registered"	when there is no registered executor for the method;

				* "executor error" when executor failed to execute method;
				
				* "couldn't deactivate object" when LogicRunner ArtifactManager failed to deactivate contract 
				after method execution; 
				
				* "couldn't update object" when LogicRunner ArtifactManager failed to update object state record;
				
				* "couldn't save results" when LogicRunner ArtifactManager failed to register results.


type LogicRunner struct

	LogicRunner struct includes fields of different types. Some of them are initialised whith the initialisation of 
	LogicRunner object. Others are sub-components, that are external objects or interfaces and dependency injection mechanism
	is used, so that these objects instances are injected into a new LogicRunner structure each time it is initialised.

	These sub-components are:

	* MessageBus                 type: core.MessageBus                 
	* ContractRequester          type: core.ContractRequester          
	* Ledger                     type: core.Ledger                     
	* NodeNetwork                type: core.NodeNetwork                
	* PlatformCryptographyScheme type: core.PlatformCryptographyScheme 
	* ParcelFactory              type: message.ParcelFactory           
	* PulseStorage               type: core.PulseStorage               
	* ArtifactManager            type: core.ArtifactManager            
	* JetCoordinator 			 type: core.JetCoordinator

	Beside this, LogicRunner structure includes fields:

	* Executors type: [core.MachineTypesLastID]core.MachineLogicExecutor
		A list of virtual machines with their MachineLogicExecutor interfaces to implement. 
	* machinePrefs type: []core.MachineType
		A list of virtual machines, where a new virtual machine is apended, when a LogicRunner component starts 
		and registers its executor.  
	* Cfg type: *configuration.LogicRunner
		Pointer to LogicRunner configuration structure. See configuration package for default LogicRunner configuration.
	* state type: map[Ref]*ObjectState
		A map type value, that implements a hash table, where ObjectState is a Value and reference to it is a Key,
		so that if LogicRunner object exists we can validate it (for details see Go map type documentation).
	* stateMutex type: sync.RWMutex
		An RWMutex struct used to prevent LogicRunner object from mutual read\write exeption 
		(for details see sync package documentation).  
	* sock type: net.Listener
		A network listener interface. 
	
