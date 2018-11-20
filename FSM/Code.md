### stateMachines.py 

```python


class StateMachine:
    def __init__(self):
        self.handlers ={}
        self.startState = None
        self.totalTraverse = []
        self.endState = []

    def add_state(self, name, handler, end_state = 0 ):
        name = name.upper()
        self.handlers[name] = handler
        if end_state:
            self.endState.append(name)

    def set_start(self,name):
        self.startState = name.upper()

    def traverse_TCP_states(self, traverse_list):
        try:
            handler = self.handlers[self.startState]

        except:
            raise Exception(".set_start() shoulbe called before .run , Mendatory")

        #종료 상태가 하나라도 없을 경우
        if not self.endState:
            raise Exception("at least one state must be an end_state")

        while True:
            (newState, traverse_list) = handler(traverse_list)
            if newState.upper() in self.endState:
                stateReturn = newState.split('_', 1)
                if stateReturn == ["ERROR"]:
                    print("TCP Terminated:", "ERROR")
                else:
                    print("TCP Terminated:", stateReturn[1])
                break
            else:
                handler = self.handlers[newState.upper()]
```

### FiniteStateMachine

```python
from stateMachines import StateMachine
import time


def CLOSED(state_list):
    if len(state_list) > 0:
        split_list = [[state_list.pop(0)], state_list]
        app, state_list = split_list

        if app[0] == "APP_ACTIVE_OPEN":
            print("create TCB\nsend:SYN")
            newState = "SYN_SENT"
        elif app[0] == "APP_PASSIVE_OPEN":
            print("create TCB\nsend: --")
            newState = "LISTEN"
        else:
            newState = "ERROR"
    #만약 위의 리스트에 아무것도 없어서 여기서 종료될 경우
    else:
        print("Process ends")
        newState = "END_CLOSED"


    return (newState, state_list)

def SYN_SENT(state_list):
    if len(state_list) > 0:
        split_list = [[state_list.pop(0)] , state_list]
        order, state_list = split_list

        if order[0] == "RCV_SYN_ACK":
            print("recv:SYN,ACK\nsend:ACK")
            newState = "ESTABLISHED"
        elif order[0] == "RCV _SYN":
            print("recv:SYN\nsend: SYN, ACK")
            newState ="SYN_RCVD"
        elif order[0] =="APP_CLOSE":
            print("delete TCB")
            newState = "CLOSED"
        else:
            newState = "ERROR"
    else :
        print("Process ends")
        newState = "END_SYN_SENT"


    return (newState, state_list)

def LISTEN(state_list):
    if len(state_list) > 0:
        split_list = [[state_list.pop(0)], state_list]
        order, state_list = split_list
        if order[0] == "RCV_SYN":
            print("recv:SYN\nsend:SYN,ACK")
            newState = "SYN_RCVD"
        elif order[0] == "APP_SEND":
            print("app:SEND\nsend:SYN")
            newState ="SYN_SENT"
        elif order[0] =="APP_CLOSE":
            print("delete TCB")
            newState = "CLOSED"
        else:
            newState = "ERROR"
    else:
        print("Process ends")
        newState = "END_LISTEN"


    return (newState, state_list)

def SYN_RCVD(state_list):
    if len(state_list) > 0:
        split_list = [[state_list.pop(0)], state_list]
        order, state_list = split_list
        if order[0] == "RCV_ACK":
            print("recv:ACK\nsend:--")
            newState = "ESTABLISHED"
        elif order[0] =="APP_CLOSE":
            print("send:FIN")
            newState = "FIN_WAIT_1"
        else:
            newState = "ERROR"
    else:
        print("Process ends")
        newState = "END_SVN_RCVD"


    return (newState, state_list)


def ESTABLISHED(state_list):
    if len(state_list) > 0:
        split_list = [[state_list.pop(0)], state_list]
        order, state_list = split_list

        if order[0] == "RCV_FIN":
            print("recv:FIN\nsend: ACK")
            newState = "CLOSE_WAIT"
        elif order[0] == "APP_CLOSE":
            print("send:FIN")
            newState = "FIN_WAIT_1"
        else:
            newState = "ERROR"
    else:
        print("Process ends")
        newState = "END_ESTABLISHED"


    return (newState, state_list)

def CLOSE_WAIT(state_list):
    if len(state_list) > 0:
        split_list = [[state_list.pop(0)], state_list]
        order, state_list = split_list

        if order[0] == "APP_CLOSE":
            print("app:CLOSE\nsend:FIN")
            newState = "LAST_ACK"

        else:
            newState = "ERROR"

    else:
        print("Process ends")
        newState = "END_CLOSE_WAIT"


    return (newState, state_list)


def LAST_ACK(state_list):
    if len(state_list) > 0:
        split_list = [[state_list.pop(0)], state_list]
        order, state_list = split_list

        if order[0] == "RCV_ACK":
            print("recv:ACK\nsend:--")
            newState = "CLOSED"

        else:
            newState = "ERROR"

    else:
        print("Process ends")
        newState = "END_LAST_ACK"


    return (newState, state_list)

def FIN_WAIT_1(state_list):
    if len(state_list) > 0:
        split_list = [[state_list.pop(0)], state_list]
        order, state_list = split_list

        if order[0] == "RCV_FIN":
            print("recv:ACK\nsend:ACK")
            newState = "CLOSING"

        elif order[0] == "RCV_FIN_ACK":
            print("recv:FIN,ACK\nsend:ACK")
            newState = "TIME_WAIT"

        elif order[0] == "RCV_ACK":
            print("recv:ACK\nsend: --")
            newState = "FIN_WAIT_2"

        else:
            newState = "ERROR"

    else:
        print("Process ends")
        newState = "END_FIN_WAIT_1"

    return (newState, state_list)

def CLOSING(state_list):
    if len(state_list) > 0:
        split_list = [[state_list.pop(0)], state_list]
        order, state_list = split_list

        if order[0] == "RCV_ACK":
            print("recv:ACK\nsend:--")
            newState = "TIME_WAIT"

        else:
            newState = "ERROR"
    else:
        print("Process ends")
        newState = "END_CLOSING"

    return (newState, state_list)

def FIN_WAIT_2(state_list):
    if len(state_list) > 0:
        split_list  =[[state_list.pop(0)], state_list]
        order, state_list = split_list

        if order[0] == "RCV_FIN":
            print("recv:FIN\nsend:ACK")
            newState = "TIME_WAIT"

        else:
            newState = "ERROR"

    else:
        print("Process ends")
        newState = "END_FIN_WAIT_2"

    return (newState, state_list)

def TIME_WAIT(state_list):
    if len(state_list) > 0:
        split_list  =[[state_list.pop(0)], state_list]
        order, state_list = split_list

        #default MSL linux kernel은 60s 따라서 2MSL = 120s

        if order[0] == "TIMEOUT_2MSL":
            time.sleep(120)
            print("Timeout=2MSL\ndelete TCB")
            newState = "CLOSED"

        else:
            newState = "ERROR"

    else:
        print("Process ends")
        newState = "END_TIME_WAIT"

    return (newState, state_list)

if __name__ ==  "__main__":
    sm = StateMachine()
    sm.add_state("Start", CLOSED)
    sm.add_state("LISTEN", LISTEN)
    sm.add_state("SYN_SENT", SYN_SENT)
    sm.add_state("SYN_RCVD", SYN_RCVD)
    sm.add_state("ESTABLISHED", ESTABLISHED)
    sm.add_state("FIN_WAIT_1",FIN_WAIT_1)
    sm.add_state("CLOSING", CLOSING)
    sm.add_state("FIN_WAIT_2", FIN_WAIT_2)
    sm.add_state("TIME_WAIT", TIME_WAIT)
    sm.add_state("CLOSE_WAIT", CLOSE_WAIT)
    sm.add_state("LAST_ACK", LAST_ACK)

    sm.add_state("ERROR", None, end_state=1)
    sm.add_state("END_CLOSED", None , end_state=1)
    sm.add_state("END_LISTEN", None, end_state=1)
    sm.add_state("END_SYN_SENT", None, end_state=1)
    sm.add_state("END_SYN_RCVD", None, end_state=1)
    sm.add_state("END_ESTABLISHED", None, end_state=1)
    sm.add_state("END_FIN_WAIT_1", None, end_state=1)
    sm.add_state("END_CLOSING", None, end_state=1)
    sm.add_state("END_FIN_WAIT_2", None, end_state=1)
    sm.add_state("END_TIME_WAIT", None, end_state=1)
    sm.add_state("END_CLOSE_WAIT", None, end_state=1)
    sm.add_state("END_LAST_ACK", None, end_state=1)

    sm.set_start("Start")
    sm.traverse_TCP_states(["APP_ACTIVE_OPEN","RCV_SYN_ACK","RCV_FIN"])
    #sm.traverse_TCP_states(["APP_PASSIVE_OPEN", "RCV_SYN","RCV_ACK"])
    #sm.traverse_TCP_states(["APP_ACTIVE_OPEN", "RCV_SYN_ACK", "RCV_FIN", "APP_CLOSE"])
    #sm.traverse_TCP_states(["APP_PASSIVE_OPEN","RCV_SYN","RCV_ACK","APP_CLOSE","APP_SEND"])
    #sm.traverse_TCP_states(["APP_ACTIVE_OPEN"])

```
