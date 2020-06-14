This short example will show how to use the SCS BDD approach for usage with Frame.

Defintion:
A play (testing suite) consists of scenes.
A scene consists of frames.
A frame consists of one or more actions
Actions in a frame can potentially run in parallel. The scene is over when all frames were played
The play is over when all scenes were played

Steps are parsed into frames based on their context. Context is provided by keywords like "Given, When, Then", "T0 Online", "T0", "Afterwards"

Validation of a When step can be postponed indefinetly provided a context is given for the Then part ("after EOD", "after Online")

Given steps of the same scenario should always be executed synchronously. So one step has to finish (sync) before the next one is executed.
However multiple scenarios inside the same frame can always be executed in parallel


Example (without datatables for brevity):

Scenario 1:

Given XFRA trades are received for T0 Online
When a corporate action MT564 is received on T1 Online
Then the corporate action event can be seen in the GUI
And a new corporate action preadvice event is created in the risk snapshot for T1


Scenario 2:

Given XETR trades are received for T0 Online
And a corporate action MT564 is received on T1 Online
When a corporate action MT564 cancellation is received on T2 Online
And afterwards a corporate action MT564 is received
Then after T2 the corporate action is marked as cancelled in the GUI


 Parsed into a playbook structure:

Scene T0 Online:
  Frame 1:  Given: [send XFRA Trades + sync, send XETR Trades + sync] run in parallel for different scenarios

Scene T1 Online:
  Frame 1:  Given: [send MT564 preadvice + sync] from scenario 2
  Frame 2:  When:  [send MT564 preadvice + sync] from scenario 1
  Frame 3:  Then: [validate corporate action event in GUI] for scenario 1

Scene T1 EOD:
   Frame 1: Then: [check snapshot for ca event]

Scene T2 Online:
   Frame 1: When: [send MT564 cancellation + sync] from scenario 2
   Frame 2: When: [send MT564 preadvice +sync] from scenario 2

Scene T2 EOD:
   Frame 1: Then: [validate corporate action event in GUI] for scenario 2


Now we have a complete test script. For each scene all frames (the given when then steps) are played until we run out of scenes and the test is over.
Imagine the following (after script was parsed):

- Create environment based on scenario description (not sure yet what kind of setup might be needed here)
- Play scene "T0 Start", which starts T0 and basically sets us into online mode. This includes sending SOD messages etc. 
- Play scene "T0 Online" 
- Play scene "T0 EoD" Send EoS triggers
- Play scene "T1 Start"
- Play scene "T1 Online"
- Play scene "T1 EoD"
- Play scene "T2 Start"
- Play scene "T2 Online"
- Play scene "T2 EoD"

After all scenes were played all events should have been fired, syncronised and validated.

Some scenes will be available by default, without explicitly mentioning them,
These are for example the Start and End scenes, which roll over to the next business day.


Potential Problems:

frames could potentially "dirty" the test context.
If a test manually sends an EOD, no other test can send an EOD:
Unless there is a rewind = taking snapshots of db and broker state before a test and re-applying it before the next one (going to the previous scene, with alternate endings),
but that will add more complexity than it might be worth it. Better to write compatible scenes
