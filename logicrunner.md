func (lr *LogicRunner) executeMethodCall(ctx context.Context, es *ExecutionState, m *message.CallMethod) (core.Reply, error)
	Executes the contract when some CallMethod initiates contract execution. 
	When execution is successfuly finished *executeMethodCall* 
	writes execution result to a *Result* field and *current.Request* data to a *Request* field 
	of the *reply.CallMethod* and returns a pointer to the *reply.CallMethod*. executeMethodCall 
	also deactivates contract object after executing the contract and registers results of operation 
	with ArtifactManager. Method has a receiver, that is a pointer to LogicRunner type.

	parameters:
		- ctx type: context.Context
		  Context type object of context package from where the currrent context data (e.g. current contract descriptors) 
		  is retrieved

		- es type: *ExecutionState
		  ExecutionState pointer for current contract object execution state
		  
		- m type: *message.CallMethod
		  Pointer to the message from initiating call method 

	Successful method execution
		return: 
			- reply.CallMethod type:    CallMethod *reply* propertie with *result* and *request* fields, where
				*result* is a result of CallMethod execution performed by LogicRunner executor and *request* is 
				the pointer to current.Request.
			- nil type: error 
			  Nil error for successful method result	
	Unsuccessful method execution
		return:
			- nil
  //---------------------------------------------------------------------------
  
  func (lr *LogicRunner) CheckOurRole(ctx context.Context, msg core.Message, role core.DynamicRole) error

	*CheckOurRole* method performes authorization, checking the role retrieved from the Context against DynamicRole, 
	and returns error if authorization failed or nill value if not. Method has a receiver, that is a pointer to 
	LogicRunner type.

	parameters:
		- ctx type: context.Context
		  Context type object of context package from where the currrent role data is retrieved

		- msg type: core.Message
		  Message type object of core package used to record authorization data to default target (see [DefaultTarget])

		- role type: core.DynamicRole
		  DynamicRole (see [DynamicRole]) object of core package to get the role data, which the current role 
		  will be checked against

	Successful authorization
		return: nil
	Unsuccessful authorization
		return:
			- error type: string Error message:
				* "authorization failed with error <error>" returnes if an error occured, while authorization, 
				where <error> is a specific error number
				* "can't execute this object" returns in case of unsuccessful authorization
        
 //----------------------------------------------------------------------------------------------------------
 
 func (lr *LogicRunner) executeActual(ctx context.Context, parcel core.Parcel, msg message.IBaseLogicMessage) (core.Reply, error) {

	ref := msg.GetReference()
	os := lr.UpsertObjectState(ref)

	os.Lock()
	if os.ExecutionState == nil {
		os.ExecutionState = &ExecutionState{
			ArtifactManager: lr.ArtifactManager,
			Queue:           make([]ExecutionQueueElement, 0),
			Behaviour:       &ValidationSaver{lr: lr, caseBind: NewCaseBind()},
		}
	}
	es := os.ExecutionState
	os.Unlock()

	// ExecutionState should be locked between CheckOurRole and
	// appending ExecutionQueueElement to the queue to prevent a race condition.
	// Otherwise it's possible that OnPulse will clean up the queue and set
	// ExecutionState.Pending to NotPending. Execute will add an element to the
	// queue afterwards. In this case cross-pulse execution will break.
	es.Lock()

	err := lr.CheckOurRole(ctx, msg, core.DynamicRoleVirtualExecutor)
	if err != nil {
		es.Unlock()
		return nil, errors.Wrap(err, "[ Execute ] can't play role")
	}

	if lr.CheckExecutionLoop(ctx, es, parcel) {
		es.Unlock()
		return nil, os.WrapError(nil, "loop detected")
	}

	request, err := lr.RegisterRequest(ctx, parcel)
	if err != nil {
		es.Unlock()
		return nil, os.WrapError(err, "[ Execute ] can't create request")
	}

	_, span := instracer.StartSpan(ctx, "LogicRunner.QueueCall")

	// Attention! Do not refactor this line if no sure. Here is no bug. Many specialists spend lots of time
	// to write it as it is.
	span.End()

	qElement := ExecutionQueueElement{
		ctx:     ctx,
		parcel:  parcel,
		request: request,
		pulse:   lr.pulse(ctx).PulseNumber,
	}

	es.Queue = append(es.Queue, qElement)
	es.Unlock()

	err = lr.StartQueueProcessorIfNeeded(ctx, es, msg)
	if err != nil {
		return nil, err
	}

	return &reply.RegisterRequest{
		Request: *request,
	}, nil
}
