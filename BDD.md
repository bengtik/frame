json -> xml
json -> protobuf
json -> gui actions (interface to selenium)
json -> swift (msif)
json -> ca events (db insert, swift)

for prototype (ca preadvice)
load trades
load events
run netting

```
Collection of implied frames can be called a Scene


Frame
|Before SOD|After SOD|Online         |Online + 1     | Before EOS|After EOS|
|          |         |trades received|trades received|           |         |

Given trades are received on T0 Online
And afterwards trades are received


When a corporate action MT564 is received on T0 Online
And afterwards a corporate action MT564 is received

Then a new corporate action preadvise event is created in the risk snapshot after EOS



Closer look at Scene Online:
 

T0
Given                 When                Then     Given               When                Then | Given   When Then
|Online                                            | Online + 1                                 | After EOS                                |
trades are received |  MT564 is received |         | trades are loaded | MT564 is received|     |        |     | event present in snapshot |


```
