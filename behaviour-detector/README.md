behaviour-detector
====================


Drift detection
------------------
Using course, heading and speed we have a simple criterion for detecting if a position report is a drift candidate. Complexity hits when we want to answer this question:

* When did the vessel **start** drifting?

To answer this question the following algorithm is proposed:

Define `E` as the maximum time between two drift detections for them to be considered as one drift path.

Define `T` as the maximum time that a vessel can stop drifting before breaking a drift path.

Before recording a drift path we require at least two drift detections on the same path.

Now let's introduce some notation that will make the algorithm much more concise to explain.

`D` is a drift detection, `N` is a non drifting report.

We now want to process a stream of position reports (any reports out of time order are chucked). Consider a small buffer that we pass reports through (turns out a size of 2 is sufficient) and sometimes drift paths come out the other side. Using the rules below we will see that the buffer only needs to hold a maximum of three elements at a time.

This is how we represent a sequence of reports and how we reference time for the reports:

&nbsp;&nbsp;&nbsp;&nbsp;D<sub>1</sub>N<sub>2</sub> represents a drift detection at time t<sub>1</sub> followed by a non drift detection at time t<sub>2</sub>.

Exiting the buffer we should only see Ds and we indicate the start of a drift path by quoting a D. Here's an example:

D'<sub>1</sub>D<sub>2</sub>D'<sub>3</sub>D<sub>4</sub>

A report that is kept in the buffer after being emitted is represented by bolding: <b>D<sub>1</sub></b>.

Note that with the buffer we also need a record of the start time of the current drift path. That start time is reset whenever a D' is emitted from the buffer.

Now suppose we have reports in the buffer. This is how the buffer is handled:

We define the following transformation rules (which are applied repeatedly till no change) for elements in the buffer:

1. N<sub>1</sub>  &#8594; nothing<br/><br/>
2. D<sub>1</sub>D<sub>2</sub>  &#8594; <br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>D<sub>2</sub></b>, emit D'<sub>1</sub>D<sub>2</sub> if t<sub>2</sub> - t<sub>1</sub> &lt; E<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;D<sub>2</sub> otherwise<br/><br/>
3. <b>D<sub>1</sub></b>D<sub>2</sub>  &#8594; <br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>D<sub>2</sub></b>, emit D<sub>2</sub> if t<sub>2</sub> - t<sub>1</sub> &lt; E<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;D<sub>2</sub> otherwise<br/><br/>
4. D<sub>1</sub>N<sub>2</sub>N<sub>3</sub> &#8594; D<sub>1</sub>N<sub>2</sub><br/><br/>
5. <b>D<sub>1</sub></b>N<sub>2</sub>N<sub>3</sub> &#8594; <b>D<sub>1</sub></b>N<sub>2</sub><br/><br/>
6. D<sub>1</sub>N<sub>2</sub>D<sub>3</sub>  &#8594; <br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;D<sub>1</sub>D<sub>3</sub>, if t<sub>3</sub> - t<sub>2</sub> &lt; T and t<sub>3</sub> - t<sub>1</sub> &lt; E<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;D<sub>3</sub> otherwise<br/><br/>
7. <b>D<sub>1</sub></b>N<sub>2</sub>D<sub>3</sub>  &#8594; <br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<b>D<sub>3</sub></b>, emit D<sub>3</sub> if t<sub>3</sub> - t<sub>2</sub> &lt; T and t<sub>3</sub> - t<sub>1</sub> &lt; E<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;D<sub>3</sub> otherwise<br/><br/>

In terms of memory utilization it should be noted that one of these buffers is required per vessel and is retained for the life of the application. Given that 30 to 40 thousand distinct vessels traverse the Australian SRR per year this may be important to optimize.