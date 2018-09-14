---
layout: post
date: 2018-09-14 6:00
title: "Event Sourced Aggregates and Error/Exception flows"
tags:
- DDD
- Event Sourcing
- Flows
- Aggregate Root
---

Recently in my team, we had a discussion about different ways of handling negative flows in aggregate roots and handling it by presentation or error handling logic. To present my point of view
I wrote this post which is comparing different approaches:

#### Aggregate which is ignoring negative flows

```ruby
class Order
  include AggregateRoot

  def initialize
    self.state = :new
    # any other code here
  end

  def submit
    return if state == :submitted
    return if state == :expired
    apply OrderSubmitted.new(data: {delivery_date: Time.now + 24.hours})
  end

  def expire
    apply OrderExpired.new
  end

  private
  attr_accessor :state

  def apply_order_submitted(event)
    self.state = :submitted
  end

  def apply_order_expired(event)
    self.state = :expired
  end
end

```

```ruby
class OrderController < ApplicationController
  def create
    dispatch_command SubmitOrder.new
    render :edit
  end
end
```

Cons:
- we are losing information about when somebody already submitted same order
- we were losing information about when somebody tried to submit an order which is already expired
- we are not able to handle any such exceptions on UI; we are ignoring it, the user is more confused

Pros:
- none


#### Aggregate which is using exceptions to handle negative flows
```ruby
class Order
  include AggregateRoot
  HasBeenAlreadySubmitted = Class.new(StandardError)
  HasExpired              = Class.new(StandardError)

  def initialize
    self.state = :new
    # any other code here
  end

  def submit
    raise HasBeenAlreadySubmitted if state == :submitted
    raise HasExpired if state == :expired
    apply OrderSubmitted.new(data: {delivery_date: Time.now + 24.hours})
  end

  def expire
    apply OrderExpired.new
  end

  private
  attr_accessor :state

  def apply_order_submitted(event)
    self.state = :submitted
  end

  def apply_order_expired(event)
    self.state = :expired
  end
end
```

```ruby
class OrderController < ApplicationController
  def create
    dispatch_command SubmitOrder.new
  rescue Order::HasBeenAlreadySubmitted
    render :edit, notice: 'Order already submitted!'
  rescue Order::HasExpired
    render :edit, notice: 'Order has expired!'
  end
end
```

Cons:
- we are losing information about when somebody already submitted same order
- we were losing information about when somebody tried to submit an order which is already expired

Pros:
- we can handle errors/negative flows/exceptions on UI
- in case of exception which is not handled in the controller, we will get nice Rollbar error


#### Aggregate which is using Events to handle negative flows
```ruby
class HasBeenAlreadySubmitted < Event
class HasExpired < Event

class Order
  include AggregateRoot

  def initialize
    self.state = :new
    # any other code here
  end

  def submit
    if state == :submitted
      apply HasBeenAlreadySubmitted.new
      return
    end
    if state == :expired
      apply HasExpired.new
      return
    end
    apply OrderSubmitted.new(data: {delivery_date: Time.now + 24.hours})
  end

  def expire
    apply OrderExpired.new
  end

  private
  attr_accessor :state

  def apply_order_submitted(event)
    self.state = :submitted
  end

  def apply_order_expired(event)
    self.state = :expired
  end
end
```

```ruby
class OrderController < ApplicationController
  def create
    event_store.within do
      dispatch_command SubmitOrder.new
    end.subscribe(to: [Order::OrderSubmited]) do
      redirect_to success_path
    end.subscribe(to: [Order::HasBeenAlreadySubmitted]) do
      render :edit, notice: 'Order already submitted!'
    end.subscribe(to: [Order::HasExpired]) do
      render :edit, notice: 'Order has expired!'
    end.call
  end
end
```

Cons:
- none (having more business specific events is a pro)

Pros:
- we know that specific user (meta) already submitted the same order, we can analyze this data, or troubleshot for customer service reason
- we know that specific user (meta) tried to submit an order which is already expired, we can analyze this data, or troubleshot for customer service reason
- we can handle errors on UI
- we can react (in event handlers) for specific events like HasExpired, by releasing booked tickets/products
back to pool/products warehouse
- we have more business specific events related to workflow (then exceptions are really exceptional - like network errors).
- additionally, we can apply multiple events in the same action, when an exception can be raised only once

#### Aggregate which is using idea of `Infra::IdempotentEventApplied` event to handle negative flows
```ruby
class Order
  include AggregateRoot

  def initialize
    self.state = :new
    # any other code here
  end

  def submit
    if state == :submitted
      apply Infra::IdempotentEventApplied.new(
        event_name: "HasBeenAlreadySubmitted",
        payload:    event.payload
      )
      return
    end
    if state == :expired
      apply Infra::IdempotentEventApplied.new(
        event_name: "HasExpired",
        payload:    event.payload
      )
      return
    end
    apply OrderSubmitted.new(data: {delivery_date: Time.now + 24.hours})
  end

  def expire
    apply OrderExpired.new
  end

  private
  attr_accessor :state

  def apply_order_submitted(event)
    self.state = :submitted
  end

  def apply_order_expired(event)
    self.state = :expired
  end
end
```

```ruby
class OrderController < ApplicationController
  def create
    event_store.within do
      dispatch_command SubmitOrder.new
    end.subscribe(to: [Order::OrderSubmited]) do
      redirect_to success_path
    end.subscribe(to: [Infra::IdempotentEventApplied]) do |event|
      case event.event_name
      when 'HasExpired'
        render :edit, notice: 'Order has expired!'
      when 'HasBeenAlreadySubmitted'
        render :edit, notice: 'Order already submitted!'
      end
    end.call
  end
end
```

Cons:
- we are introducing artificial event which not belongs to the domain (it will not be in events directory)
- we are not using types which are the best way to describe our business process (negative flows are still business flows)
- we can't find an easy way such events for specific use case
- we are forced to dispatch based on strings
- we have fewer business events defined (so some important part of the business process is ignored)

Pros:
- we know that specific user (meta) already submitted the same order, we can analyze this data, or troubleshot for customer service reason
- we know that specific user (meta) tried to submit an order which is already expired, we can analyze this data, or troubleshot for customer service reason
- we can handle errors on UI
- we can react (in event handlers) for specific events like HasExpired, by releasing booked tickets/products
back to pool/products warehouse

# Summary
- Exceptions which are not system based (network error) are the best candidates for events
- An exception like PaymentGatewayFailed can be converted to a valuable event which can be handled by our infrastructure (sending emails/slacks to ops team, an operation can be scheduled for retry etc...)
- Negative flows are still business use cases
- Ignoring negative flows is damaging user flow, messing with his expectations, leading to bad design (maybe your Aggregates are too much CRUD'y, but event then in CRUD you would like to react for
active record validations and display them to the user), limiting us with ways of handling them (event handlers, sagas)
- If we would like to be data-driven there is no better way than tracking events in events oriented architecture
- How having more events (and having more insights about user and system behaviors) can be a bad thing? Such events are not useless lines of code, they are adding more meaning to the domain.