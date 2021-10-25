Event Downsampler
=================

![model](./assets/model.png)

This FSM aims to downsample events in a stream. To this end, it receives as input the counter of the events and yields as output
a bunch of information including when to print a message reporting on the aggregate of the number of events tha have occurred.

The FSM is composed of two parallel charts:

- `EvalOccurence` is devoted to evaluating the number of events triggered in a given temporal window (default = 1 sec).
- `EventHandler` is the actual handler implementing the logic below:
  - If the occurrence of the events is above a threshold, then it triggers an output message only each second (i.e., same lapse as above).

We have only 2 params:
- The temporal window is used to evaluate the frequency of the input events and to carry out down-sampling at the same time.
- The threshold for the events detected in the window above, which triggers down-sampling (default = 5).

The output of the handler is threefold:
- The boolean `DOWNSAMPLE` that is `1` when the handler is downsampling the input events. 
- The boolean `TRIGGER` that tells when to print the message.
- The integer `CNT_OUT` that accounts for the number of events that occurred since the last print (this info can be used to populate the message).

Here's below a typical outcome:

![graph](./assets/graphs.png)

Find below a meta-code that can be used to interface the handler for managing the occurrence of multiple event types:

```c++
if (event_1) {
  CNT_IN++;
  if (!FSM.DOWNSAMPLE) {
    yInfo() << "event 1 detected";
  }
}

if (event_2) {
  CNT_IN++;
  if (!FSM.DOWNSAMPLE) {
    yInfo() << "event 2 detected";
  }
}

// ...

FSM_step(CNT_IN);

if (FSM.DOWNSAMPLE && FSM.TRIGGER) {
  yInfo() << "Detected <<" FSM.CNT_OUT << "events on aggregate since the last message";
}
```

### References
This model has been developed to address https://github.com/robotology/icub-main/issues/768.
