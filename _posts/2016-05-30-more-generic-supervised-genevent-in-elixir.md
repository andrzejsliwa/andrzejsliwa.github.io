---
layout: post
date: 2016-05-30 20:34
title: "More generic supervised GenEvent in Elixir"
category: 
- programming
- system design
tags:
- GenEvent
- Elixir
---
Some Elixir developers wonder how to supervise the **GenEvent** properly. Some of them waiting for incoming **GenBroker**&nbsp; ;). Important fact about GenEvent implementation is that handlers are not separate processes, which leads to problem with supervision. Following the elixir [documentation](http://elixir-lang.org/docs/stable/elixir/GenEvent.html#add_mon_handler/3) you can find really useful function (available only in Elixir as extension of standard OTP behaviour):

```elixir
add_mon_handler(manager, handler, term)
```
Based on [documentation](http://elixir-lang.org/docs/stable/elixir/GenEvent.html#add_mon_handler/3) you will find that its adds a monitored event handler to the event manager. In case of failure event handler will be deleted and event manager will send a message to the calling process:

```elixir
{:gen_event_EXIT, handler, reason} 
```
Documentation describe also important fact that mentioned message is not guaranteed to be delivered in case the manager crashes. So If you want to guarantee the message is delivered, you have two options:

* **monitor the event manager**
* link to the event manager and then set **Process.flag(:trap_exit, true)** in your handler callback

Lets take a look closer on first approach. But before we will do so I would like to focus on defining two simple (and identical for purpose of example) events handlers:

```elixir
    defmodule Cqrs.CommandHandler do
      use GenEvent

      def handle_event(:ok, state) do
        {:ok, state}
      end
    end
```
and
```elixir
    defmodule Cqrs.EventHandler do
      use GenEvent

      def handle_event(:ok, state) do 
        {:ok, state}
      end
    end
```
Lets assume that both handlers based on his names will have different purposes, but both will **subscribe/listen** for events from same **event manager**. I’m skipping implementation of those handlers for simplicity reason. For purpose of this example lets also introduce simple (and almost empty) Event Manager:
```elixir
    defmodule Cqrs.Bus do
      def start_link() do
        GenEvent.start_link(name: __MODULE__)
      end
    end
```
Like you see Event Manager is just a named process.

The first step in case of monitored Event Manager is implementing simple Watcher process. But wait, lets go little bit further, and try to implement it in generic way.

I strongly following TDD approach so lets write first very simple test case for it:
```elixir
    defmodule Cqrs.HandlerWatcherTest do
      use ExUnit.Case

      alias Cqrs.HandlerWatcher

      defmodule ExampleHandler do
        use GenEvent

        def init(%{callback_pid: pid} = state) do
          send pid, :init_called
          {:ok, state}
        end
     
        def handle_event({:expected}, %{callback_pid: pid} = state) do
          send pid, :handle_event_called
          {:ok, state}
        end
      end

      setup do
        name = __MODULE__
        {:ok, _pid} = GenEvent.start_link(name: name)
        {:ok, name: name}
      end
    end
```
Like you see above, I defined simple ExampleHandler for testing purpose which I will use to verify implementation.

Here you will find basic test cases for my **HandlerWatcher**:
```elixir
     test “Forwards event to named process and handle it”,
       %{name: name} do
       assert {:ok, _pid} =
         HandlerWatcher.start_link(name, ExampleHandler, %{callback_pid: self})

       assert_receive :init_called
       assert :ok = GenEvent.notify(name, {:expected})
       assert_receive :handle_event_called
     end

     @tag capture_log: true
     test “handler was re-added automatically”,
       %{name: name} do
       assert {:ok, _pid} =
         HandlerWatcher.start_link(name, ExampleHandler, %{callback_pid: self})

       assert_receive :init_called
       assert :ok = GenEvent.notify(name, {:wrong})
       assert_receive :init_called
       assert :ok = GenEvent.notify(name, {:expected})
       assert_receive :handle_event_called
     end

    test “handler was removed and watcher process stoped”,
      %{name: name} do
      Process.flag(:trap_exit, true)
      assert {:ok, pid} =
        HandlerWatcher.start_link(name, ExampleHandler, %{callback_pid: self})

      assert_receive :init_called
      assert :ok = 
        HandlerWatcher.remove_handler(ExampleHandler, %{callback_pid: self})
      assert_receive {:EXIT, ^pid, :normal}
    end 
```
Like you see in those examples I’m using message passing to verify my implementation. Worth of mentioning here is also using 
**capture_log** which lets you hide expected errors output of specific test function. In case of problem with understanding how tests are working I refering you to [ExUnit](http://elixir-lang.org/docs/stable/ex_unit/ExUnit.html) documentation.

Ok, lets now take a look on our implementation of **HandlerWatcher**:
```elixir
    defmodule Cqrs.HandlerWatcher do
      use GenServer

      defmodule State do
        defstruct handler: nil, args: [], event_manager: nil, monitor_ref: nil
      end

      def start_link(event_manager, handler, args \\ []) do
        GenServer.start_link(__MODULE__,
          [event_manager, handler, args], name: handler)
      end

      def remove_handler(handler, args) do
        GenServer.cast(handler, {:remove_handler, handler, args})
      end

      def init([event_manager, handler, args]) do
        monitor_ref = Process.monitor(event_manager)
        state = %State{event_manager: event_manager,
                       handler: handler,
                       args: args}
        {:ok, ^event_manager} =
          start_handler(state)
        {:ok, %State{state|monitor_ref: monitor_ref}}
      end

      def handle_cast({:remove_handler, handler, args},
        %State{event_manager: event_manager,
               monitor_ref: monitor_ref,
               handler: handler,
               args: args} = state) do
        :ok = GenEvent.remove_handler(event_manager, handler, args)
        Process.demonitor(monitor_ref)
        {:stop, :normal, state}
      end

      def handle_info({:DOWN, _ref, :process, {_event_manager, _node}, _reason}, state) do
        {:stop, :event_manager_down, state}
      end

      def handle_info({:gen_event_EXIT, handler, _reason},
        %State{event_manager: event_manager, handler: state_handler} =   state) do
        ^state_handler = handler
        {:ok, ^event_manager} = start_handler(state)
        {:noreply, state}
      end

      defp start_handler(
        %State{event_manager: event_manager,
               handler: handler,
               args: args}) do
        case GenEvent.add_mon_handler(event_manager, handler, args) do
          :ok -> {:ok, event_manager}
          {:error, reason} -> {:stop, reason}
        end
      end
    end
```
Purpose of this process is:

* monitoring event manager
* adding monitor handler in new process
* reacting for **{:gen_event_EXIT, handler, _reason}** with “restarting” handler
* reacting for **{:DOWN, _ref, :process, {_event_manager, _node}, _reason}** with stoping process.

In this case I’m using **defstruct** for keeping server state. In erlang I used always records for such purpose. 
Problem with structs is that I can’t reuse pattern matching of payload and particular fields of defstruct. 
For all people interested in I recomend to take a look on **__struct__** field and **[Kernel.struct/2](http://elixir-lang.org/docs/stable/elixir/Kernel.html#struct/2)** function

So far, so simple. Now it’s time for supervisor:
```elixir
    defmodule Cqrs.HandlerWatcher.Supervisor do
      use Supervisor

      defmodule Spec do
        import Supervisor.Spec, warn: false

        def gen_event_supervisor(name, event_handlers \\ []) do
          supervisor(Cqrs.HandlerWatcher.Supervisor, [name, event_handlers])
        end

        def event_handler(name, args \\ []) do
          {name, args}
        end
      end

      def start_link(event_manager, handlers) do
        Supervisor.start_link(__MODULE__, [event_manager, handlers], name: __MODULE__)
      end

      def init([event_manager, handlers]) do
        handlers = for {handler, args} <- handlers do
                     worker(Cqrs.HandlerWatcher, [event_manager, handler, []], id: handler, restart: :transient)
                   end

        children = [worker(event_manager, [])|handlers]
        supervise(children, [strategy: :one_for_one])
      end
    end
```
The purpose of this supervisor is simple, supervise event manager and all handlers. I made here also few steps forwards to make it more generic. Take a look on simple helpers of **Spec** module. I skipped all type definitions to not overload examples and to show my intent.

To see benefit of this approach lets me show you root application supervisor:
```elixir
    defmodule Cqrs do
      use Application

      def start(_type, _args) do
        import Supervisor.Spec, warn: false
        import Cqrs.HandlerWatcher.Supervisor.Spec

        children = [
          gen_event_supervisor(Cqrs.Bus,
            [event_handler(Cqrs.EventHandler, []),
             event_handler(Cqrs.CommandHandler, [])]),

          # ...
          supervisor(Cqrs.Repo, [])
        ]

        opts = [strategy: :one_for_one, name: Cqrs.Supervisor]
        Supervisor.start_link(children, opts)
      end

      # ...
    end 
```
Running this supervisor should result with such supervisor tree:

![](/assets/images/1-no1dRZBZCCKZHJvdljdncQ.png)

Main point here is reusing HandlerWatcher implementation for different handlers in order to avoid code duplication. 
Some Erlang/Elixir developers just re-implementing from scratch same functionality of watcher for each handler.
I strongly believe that current OTP is not closed subset of patterns, there is a lots of space to extend it. 
(please take a look on **[erlangpatterns.com](http://www.erlangpatterns.org/patterns.html)** introduced by **[Garret Smith](http://www.gar1t.com/about.html)**)

I encourage you to search common patterns in your code, but on refactor step for sure. First implementation should be simple and readable, which lets you see those repetitive patterns.

I’m leaving to you evaluation of this solution. Especially by killing/crashing **Cqrs.Bus** event manager, **GenEvent** handlers watchers or by sending events with unhandled payload.
