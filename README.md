# FlowChart
  
  - Flowchart is a rubygem for StateFlow(state-action-transition) and WorkFlow(user-assigned-assigner) workflow.
  - Flowchart(Stateflow) works out of the box both for ActiveRecord and Non-ActiveRecord Models.
  - Flowchart(Workflow) works out of the box both for ActiveRecord
  - It provides an easy DSL to create a workflow for your model's object.
  - State Flow + Work FLow = Process Flow
  - You can not only define states-action-transitions but also do a lot more with pre-process and post-process steps
  - You can independently use either of StateFlow and WorkFlow or together, the way you need it

## INSTALLATION

  - gem install flowchart

  - No other gem dependencies! Tested on Ruby 1.8.7 and Ruby 1.9.3 | Rails 3.2.13 and Rails 2.3.8

## SYNTAX & OPTIONS


### STATEFLOW only

```ruby
  flowchart do
    init_flowstate :init

    flowstate :init do
      preprocess Proc.new { |o| p "blah blah" }
      postprocess :some_method
    end

    flowstate :uploaded do
        preprocess Proc.new  :do_something
        postprocess Proc.new  { |o| p "File has been uploaded" }
    end

    action :upload do
      transitions :from => :init, :to => :uploaded, :condition => :file_parsable?
    end
  end
```


#### OPTIONS

- By Default flowchart assumes that the model attribute it works on is "process_status. 
If you want to override it then

```ruby
flowchart do
  state_column :some_other_column
end
```
- Every flowstate can have options preprocess and postprocess blocks. 
You can mention any instance methods or Procs/lamdas to execute.

- In Actions you can mention :condition with evaluate truth or falsity based on which the 
transition action(:from -> :to) is possible or not is determined. 
If :condition is not given transition is always evaluated to truth.

- In Actions you can also mention :branch, when you have multiple :to states from one single :from state 
then you can mention a ":branch" condition which you can execute and determine 
which branch state will be the next! Useful in decision trees.

#### Methods

- obj.{state_name}? returns boolean true/false if object is in that that state

- obj.{action} returns the transtion to next state on executing the action

- obj.{action}! returns the transition to next state on executing the action and save after call

- obj.current_state returns the current_state of the object

```ruby
  e.g  #<FlowChart::State:0x9e6e3f4 @flowstatename=:uploaded, @before_and_after_works={:postprocess=>#<Proc:0x099dc28c@(irb):30>, :preprocess=>#<Proc:0x099dc3e0@(irb):29>}>
```

- obj.flowprocessor returns the complete tree of flowchart for the object!

```ruby
  e.g. #<FlowChart::FlowProcessor:0x9e6e5e8
       @actions=
        {:close=>
          #<FlowChart::Action:0x9e6df80
           @flowprocessor=#<FlowChart::FlowProcessor:0x9e6e5e8 >,
           @name=:close,
           @transitions=
            [#<FlowChart::Transition:0x9e6dea4
              @branch=nil,
              @condition=:file_close?,
              @from=[:init, :uploaded, :open],
              @to=:closed>]>,
         :upload=>
          #<FlowChart::Action:0x9e6e110
           @flowprocessor=#<FlowChart::FlowProcessor:0x9e6e5e8 >,
           @name=:upload,
           @transitions=
            [#<FlowChart::Transition:0x9e6e084
              @branch=nil,
              @condition=:file_parsable?,
              @from=[:init],
              @to=:uploaded>]>,
         :process=>
          #<FlowChart::Action:0x9e6e070
           @flowprocessor=#<FlowChart::FlowProcessor:0x9e6e5e8 >,
           @name=:process,
           @transitions=
            [#<FlowChart::Transition:0x9e6df94
              @branch=:choose_branch,
              @condition=nil,
              @from=[:uploaded],
              @to=[:open, :closed]>]>},
       @flowstates=
        {:init=>
          #<FlowChart::State:0x9e6e4f8
           @before_and_after_works=
            {:postprocess=>:notify_user, :preprocess=>#<Proc:0x099dca84@(irb):24>},
           @flowstatename=:init>,
         :open=>
          #<FlowChart::State:0x9e6e2dc
           @before_and_after_works={:postprocess=>#<Proc:0x099db8f0@(irb):35>},
           @flowstatename=:open>,
         :uploaded=>
          #<FlowChart::State:0x9e6e3f4
           @before_and_after_works=
            {:postprocess=>#<Proc:0x099dc28c@(irb):30>,
             :preprocess=>#<Proc:0x099dc3e0@(irb):29>},
           @flowstatename=:uploaded>,
         :closed=>
          #<FlowChart::State:0x9e6e1d8
           @before_and_after_works=
            {:postprocess=>:notify_user, :preprocess=>#<Proc:0x099daf18@(irb):39>},
           @flowstatename=:closed>},
       @starting_flowstate=
        #<FlowChart::State:0x9e6e4f8
         @before_and_after_works=
          {:postprocess=>:notify_user, :preprocess=>#<Proc:0x099dca84@(irb):24>},
         @flowstatename=:init>,
       @starting_flowstate_name=:init,
       @state_column=:process_status>
```    


### WORKFLOW only

```ruby
  # Example of a Non-ActiveRecord Class using FlowChart in Ruby - showing only Work Machine
  # Require 'active_record'
  class User
    attr_accessor :email
    def initialize(email)
      @email=email
    end
  end
  u1=User.new("1@gmail.com")
  u2=User.new("2@gmail.com")
  u3=User.new("3@gmail.com")
  u4=User.new("4@gmail.com")

  class SampleWork
    include FlowChart
    require 'active_record'
    attr_accessor :assigned_to,:assigned_by
    workchart do

      assigned_to_column :assigned_to
      assigned_by_column :assigned_by

      workowner :user do
        goal_time lambda{ 2.days }
        dead_line lambda{ 3.days }
      end

      delegate :feed_pairing_work
      delegate :feed_publishing_work
      delegate :confirmation_work
    end

  end
```

#### OPTIONS

- By Default flowchart assumes that the model attributes for work_assignments are 'assigned_to' & assigned_by'
If you want to override it then

  ```ruby
    flowchart do
      assigned_to_column :some_other_column1
      assigned_by_column :some_other_column2
    end
  ```
  

- Every Workchart belongs to a workowner, here it is User Model. goal_time and dead_line are default SLA you can mention
for the work! You can call these procs later to monitor the SLA in your Application.

  ```ruby
    workowner :user do
      goal_time lambda{ 2.days }
      dead_line lambda{ 3.days }
    end
  ```

- There can be multiple tasks associated with a work. Each task here is knows as 'delegate'

  ```ruby
    delegate :feed_pairing_work
  ```

- Once the object of SampleWork is created you can use your delegate tasks and assign users to them

  ```ruby
   obj.feed_pairing_work(u1,u2)
   #or obj.feed_pairing_work!(u1,u2) mentioning ! will save the obj object after callbacks
  ```

- We cav have multiple delegates(tasks) which means each of these tasks can have completely separate set of WorkOwners(assigned_by, assigned_to). At any given point 'obj.current_work_owners' should give the work_owners at the moment for each delegates!

- If you want to see the trace of WorkOwners for each delegates go through the tree 'obj.workprocessor'. You will see for every delegate actions we keep the trace of work_owners which changes i.e yesterday it might be u1,u2 today it might be u3,u4 for a delegate. Keeping trace can be an added feature that can be used for logging in your Application.


#### Methods

- obj.{current_work_owners} returns the set of current_work_owners for each delegate task

```ruby
  e.g. [{:feed_pairing_work=>{:assigned_to=>#<User:0x99e3d84 @email="1@gmail.com">, :assigned_by=>#<User:0x9a027d4 @email="2@gmail.com">}}, {:feed_publishing_work=>{}}, {:confirmation_work=>{}}]
```

- YAML.load(obj.assigned_by) returns the set of work_owners(assiged_by) for each delegate task

```ruby
  e.g. [{:feed_pairing_work=>#<User:0x9eac744 @email="2@gmail.com">}, {:feed_publishing_work=>""}, {:confirmation_work=>""}]
```

- YAML.load(obj.assigned_to) returns the set of work_owners(assiged_to) for each delegate task

```ruby
  e.g. [{:feed_pairing_work=>#<User:0x9ea9db4 @email="1@gmail.com">}, {:feed_publishing_work=>""}, {:confirmation_work=>""}]
```

- obj.{delegate}(user1obj,user2obj) assigns the delegate task where 'user2obj' is assigner and user1.obj is assigned_to

- obj.workprocessor.work_owners returns the Hash of owner types for this work item with sla configs

```ruby
  e.g. {:user=>#<FlowChart::WorkOwner:0x9e6d760 @sla={:goal_time=>#<Proc:0x099d86b4@(irb):63>, :dead_line=>#<Proc:0x099d859c@(irb):64>}, @work_owner_model=User>}
```

- obj.workprocessor.delegate_actions returns all the delegates actions for this work that has been defined

```ruby
  e.g. {:feed_pairing_work=>
        #<FlowChart::WorkDelegateAction:0x9e6d404
         @current_work_owners=
          {:assigned_to=>#<User:0x99e3d84 @email="1@gmail.com">,
           :assigned_by=>#<User:0x9a027d4 @email="2@gmail.com">},
         @delegate_action_name=:feed_pairing_work,
         @work_owners_trace=
          [{:assigned_to=>#<User:0x99e3d84 @email="1@gmail.com">,
            :assigned_by=>#<User:0x9a027d4 @email="2@gmail.com">}]>,
       :feed_publishing_work=>
        #<FlowChart::WorkDelegateAction:0x9e6d3f0
         @current_work_owners={},
         @delegate_action_name=:feed_publishing_work,
         @work_owners_trace=[]>,
       :confirmation_work=>
        #<FlowChart::WorkDelegateAction:0x9e6d3b4
         @current_work_owners={},
         @delegate_action_name=:confirmation_work,
         @work_owners_trace=[]>}
```

- obj.workprocessor returns the complete tree of flowchart for the object!

```ruby
  e.g. #<FlowChart::WorkProcessor:0x9e6d814
         @assigned_by_column=:assigned_by,
         @assigned_to_column=:assigned_to,
         @delegate_actions=
          {:feed_pairing_work=>
            #<FlowChart::WorkDelegateAction:0x9e6d404
             @current_work_owners=
              {:assigned_to=>#<User:0x99e3d84 @email="1@gmail.com">,
               :assigned_by=>#<User:0x9a027d4 @email="2@gmail.com">},
             @delegate_action_name=:feed_pairing_work,
             @work_owners_trace=
              [{:assigned_to=>#<User:0x99e3d84 @email="1@gmail.com">,
                :assigned_by=>#<User:0x9a027d4 @email="2@gmail.com">}]>,
           :feed_publishing_work=>
            #<FlowChart::WorkDelegateAction:0x9e6d3f0
             @current_work_owners={},
             @delegate_action_name=:feed_publishing_work,
             @work_owners_trace=[]>,
           :confirmation_work=>
            #<FlowChart::WorkDelegateAction:0x9e6d3b4
             @current_work_owners={},
             @delegate_action_name=:confirmation_work,
             @work_owners_trace=[]>},
         @work_owners=
          {:user=>
            #<FlowChart::WorkOwner:0x9e6d760
             @sla=
              {:goal_time=>#<Proc:0x099d86b4@(irb):63>,
               :dead_line=>#<Proc:0x099d859c@(irb):64>},
             @work_owner_model=User>}>
```


### WORKFLOW & STATEFLOW

```ruby
# Example of a Non-ActiveRecord Class using FlowChart in Ruby - showing State Machine & Work Machine = Flowchart
# requires 'active_record' for Workflow
class User
  attr_accessor :email
  def initialize(email)
    @email=email
  end
end
u1=User.new("1@gmail.com")
u2=User.new("2@gmail.com")
u3=User.new("3@gmail.com")
u4=User.new("4@gmail.com")

class SampleFlowchart
  include FlowChart
  require 'active_record'
  attr_accessor :process_status
  attr_accessor :assigned_to,:assigned_by

  flowchart do
    init_flowstate :init

    flowstate :init do
      preprocess Proc.new { |o| p "Initializing File" }
      postprocess :notify_user
    end

    flowstate :uploaded do
      preprocess Proc.new  { |o| p "Validating File" }
      postprocess Proc.new  { |o| p "File has been uploaded in system" }
    end

    flowstate :open do
      postprocess :notify_user
      postprocess Proc.new  { |o| p "File has been closed" }
    end

    flowstate :closed do
      preprocess Proc.new  { |o| p "File closed" }
      postprocess :notify_user
    end

    action :upload do
      transitions :from => :init, :to => :uploaded, :condition => :file_parsable?
    end

    action :process do
      transitions :from => :uploaded, :to => [:open, :closed], :branch => :choose_branch
    end

    action :close do
      transitions :from => [:init,:uploaded,:open], :to => :closed, :condition => :file_close?
    end

  end

  workchart do

    assigned_to_column :assigned_to
    assigned_by_column :assigned_by

    workowner :user do
      goal_time lambda{ 2.days }
      dead_line lambda{ 3.days }
    end

    delegate :feed_pairing_work
    delegate :feed_publishing_work
    delegate :confirmation_work
  end

  def choose_branch
    6 > 5 ? :open : :closed
  end

  def notify_user
    p "Notifying User!"
  end

  def file_parsable?
    3 > 2 ? true : false
  end

  def file_close?
    1 > 0 ? true : false 
  end
end
```

## Samples
You can refer to the example codes mentioned here in 'samples' directory

## Save! in ActiveRecord Model

For any action in Stateflow and any delgate action in Work Flow if you choose to follow it up with a '!' then the model object is saved after action call. Similarly for an action in Stateflow and any delgate action in Work Flow if you choose to not follow it up with a '!' it will only update the attributes for the model object, not save it.