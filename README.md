## State machine
### NORMAL
This stable state indicates that worker is waiting for refresh() request, transition to REFRESHING right after receive refresh().
### DEAD
This is also a stable state, it indicates that the worker failed to refresh the token, there is nothing that user can do other than call restart(), it reject any refresh() call.
### REFRESHING
worker is trying to refresh token by sending a request to api server. This is an unstable state, mean that worker should not stay here too long. If the call to api server is successed, worker will have an new pair access token and refresh token, it should transition to JUST_REFRESHED state. Otherwise, if the call failed, worker will transition to DEAD state.
Note that the call may fail because of another worker (on other tabs) have refreshed the token before this worker. In this case, new tokens are already been presisted in local storage. Worker must transition to JUST_REFRESHED state.
### JUST REFRESHED
this state indicates that worker have just refresh the tokens. The tokens is new and 'hot'. Any refresh() call will return immediately with these new tokens. After 10s, the tokens are 'cold' and might expired. Worker should transitino to NORMAL state so user can replace them with new tokens.
```
                                         restart()
                      +-------------------------------------------------+
                      |                                                 |
                      |                                                 |
                +-----v----+           +--------------+            +----+---+
                |          | refresh() |              |   failed   |        |   refresh()
refresh()+------>  NORMAL  +----------->  REFRESHING  +------------>  DEAD  <-----+
                |          |           |              |            |        |
                +-----^----+           +-------+------+            +----^---+
                      |                        |                        |
                      |                        |success                 +
                      |                        |                     restart()
                      |               +--------v---------+
                      |      pass 10s |                  |
                      +---------------+  JUST_REFRESHED  |
                                      |                  |
                                      +--------^---------+
                                               |
                                               +
                                            refresh()


```