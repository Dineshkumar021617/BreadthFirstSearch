# Breadth First Search
## AIM

To develop an algorithm to find the route from the source to the destination point using breadth-first search.

## THEORY
Explain the problem statement

## DESIGN STEPS

### STEP 1:
Identify a location in the google map:

### STEP 2:
Select a specific number of nodes with distance

### STEP 3:
Import required packages.

### Step 4:
Include each node and its distance separately in the dictionary data structure.

### Step 5:
End of program.

## ROUTE MAP:
![Map](https://user-images.githubusercontent.com/75234807/166142528-9743e76e-7271-4562-ae82-c8d0093b1b0a.png)

## PROGRAM
%matplotlib inline
import matplotlib.pyplot as plt
import random
import math
import sys
from collections import defaultdict, deque, Counter
from itertools import combinations
# Prepared by 
# C. Obed Otto, 
# Department of Artificial Intelligence and Datascience,
# Saveetha Engineering College. 602105. India./
# Experiment done by
# Student name : Dineshkumar S
class Problem(object):
    """The abstract class for a formal problem. A new domain subclasses this,
    overriding `actions` and `results`, and perhaps other methods.
    The default heuristic is 0 and the default action cost is 1 for all states.
    When yiou create an instance of a subclass, specify `initial`, and `goal` states 
    (or give an `is_goal` method) and perhaps other keyword args for the subclass."""

    def __init__(self, initial=None, goal=None, **kwds): 
        self.__dict__.update(initial=initial, goal=goal, **kwds) 
        
    def actions(self, state):        
        raise NotImplementedError
    def result(self, state, action): 
        raise NotImplementedError
    def is_goal(self, state):        
        return state == self.goal
    def action_cost(self, s, a, s1): 
        return 1
    
    def __str__(self):
        return '{0}({1}, {2})'.format(
            type(self).__name__, self.initial, self.goal)
class Node:
    "A Node in a search tree."
    def __init__(self, state, parent=None, action=None, path_cost=0):
        self.__dict__.update(state=state, parent=parent, action=action, path_cost=path_cost)

    def __str__(self): 
        return '<{0}>'.format(self.state)
    def __len__(self): 
        return 0 if self.parent is None else (1 + len(self.parent))
    def __lt__(self, other): 
        return self.path_cost < other.path_cost
failure = Node('failure', path_cost=math.inf) # Indicates an algorithm couldn't find a solution.
cutoff  = Node('cutoff',  path_cost=math.inf) # Indicates iterative deepening search was cut off.
def expand(problem, node):
    "Expand a node, generating the children nodes."
    s = node.state
    for action in problem.actions(s):
        s1 = problem.result(s, action)
        cost = node.path_cost + problem.action_cost(s, action, s1)
        yield Node(s1, node, action, cost)
        

def path_actions(node):
    "The sequence of actions to get to this node."
    if node.parent is None:
        return []  
    return path_actions(node.parent) + [node.action]


def path_states(node):
    "The sequence of states to get to this node."
    if node in (cutoff, failure, None): 
        return []
    return path_states(node.parent) + [node.state]
FIFOQueue = deque
def breadth_first_search(problem):
    "Search shallowest nodes in the search tree first."
    node = Node(problem.initial)
    if problem.is_goal(problem.initial):
        return node
    # Remove the following comments to initialize the data structure
    frontier = FIFOQueue([node])
    reached = {problem.initial}
    while frontier:
        node = frontier.pop()
        for child in expand(problem, node):
            s = child.state
            if problem.is_goal(s):
                return child
            if s not in reached:
                reached.add(s)
                frontier.appendleft(child)
    return failure
class RouteProblem(Problem):
    """A problem to find a route between locations on a `Map`.
    Create a problem with RouteProblem(start, goal, map=Map(...)}).
    States are the vertexes in the Map graph; actions are destination states."""
    
    def actions(self, state): 
        """The places neighboring `state`."""
        return self.map.neighbors[state]
    
    def result(self, state, action):
        """Go to the `action` place, if the map says that is possible."""
        return action if action in self.map.neighbors[state] else state
    
    def action_cost(self, s, action, s1):
        """The distance (cost) to go from s to s1."""
        return self.map.distances[s, s1]
    
    def h(self, node):
        "Straight-line distance between state and the goal."
        locs = self.map.locations
        return straight_line_distance(locs[node.state], locs[self.goal])
class Map:
    """A map of places in a 2D world: a graph with vertexes and links between them. 
    In `Map(links, locations)`, `links` can be either [(v1, v2)...] pairs, 
    or a {(v1, v2): distance...} dict. Optional `locations` can be {v1: (x, y)} 
    If `directed=False` then for every (v1, v2) link, we add a (v2, v1) link."""

    def __init__(self, links, locations=None, directed=False):
        if not hasattr(links, 'items'): # Distances are 1 by default
            links = {link: 1 for link in links}
        if not directed:
            for (v1, v2) in list(links):
                links[v2, v1] = links[v1, v2]
        self.distances = links
        self.neighbors = multimap(links)
        self.locations = locations or defaultdict(lambda: (0, 0))

        
def multimap(pairs) -> dict:
    "Given (key, val) pairs, make a dict of {key: [val,...]}."
    result = defaultdict(list)
    for key, val in pairs:
        result[key].append(val)
    return result


Home_nearby_locations = Map(
    {('Kundrathur', 'Home'): 6, ('Kundrathur', 'Pammal'): 6,
     ('Home', 'Porur'): 4, ('Pammal', 'Airport'):5,
     ('Porur', 'Vadapalani'): 7, ('Porur', 'Maduravoyal'): 4, ('Porur', 'Guindy'): 10, ('Airport', 'Guindy'): 9,
     ('Vadapalani', 'T.Nagar'): 4, ('Vadapalani', 'Koyambedu'): 4, ('Maduravoyal', 'Koyambedu'): 5, ('Maduravoyal', 'Ambattur'): 6, ('Guindy', 'Saidapet'): 2,
     ('T.Nagar', 'EA Mall'): 5, ('Koyambedu', 'Korattur'): 6, ('Ambattur', 'Madhavaram'): 13, ('Saidapet', 'T.Nagar'): 4,
     ('EA Mall', 'Broadway'): 5, ('Korattur', 'Mahavaram'): 10, ('Madhavaram', 'Manali'): 11,
     ('Broadway', 'Tondiarpet'): 5, ('Broadway', 'Perambur'): 9, ('Manali', 'Tiruvottiyur'): 7,
     ('Tondiarpet', 'Tiruvottiyur'): 6, ('Perambur', 'Madhavaram'): 5})


r0 = RouteProblem('Home', 'Tiruvottiyur', map=Home_nearby_locations)
r1 = RouteProblem('Vadapalani', 'Tiruvottiyur', map=Home_nearby_locations)
r2 = RouteProblem('Guindy', 'Madhavaram', map=Home_nearby_locations)
r3 = RouteProblem('Kundrathur', 'Manali', map=Home_nearby_locations)
r4 = RouteProblem('Perambur', 'Home', map=Home_nearby_locations)


goal_state_path_0=breadth_first_search(r0)
goal_state_path_1=breadth_first_search(r1)
goal_state_path_2=breadth_first_search(r2)
goal_state_path_3=breadth_first_search(r3)
goal_state_path_4=breadth_first_search(r4)
print("GoalStateWithPath:{0}".format(goal_state_path_0))
print("Total Distance={0} Kilometers".format(goal_state_path_0.path_cost))
print("Route:{0}".format(path_states(goal_state_path_0)))

print("\nGoalStateWithPath:{0}".format(goal_state_path_1))
print("Total Distance={0} Kilometers".format(goal_state_path_1.path_cost))
print("Route:{0}".format(path_states(goal_state_path_1)))

print("\nGoalStateWithPath:{0}".format(goal_state_path_2))
print("Total Distance={0} Kilometers".format(goal_state_path_2.path_cost))
print("Route:{0}".format(path_states(goal_state_path_2)))
print("\nGoalStateWithPath:{0}".format(goal_state_path_3))
print("Total Distance={0} Kilometers".format(goal_state_path_3.path_cost))
print("Route:{0}".format(path_states(goal_state_path_3)))
print("\nGoalStateWithPath:{0}".format(goal_state_path_4))
print("Total Distance={0} Kilometers".format(goal_state_path_4.path_cost))
print("Route:{0}".format(path_states(goal_state_path_4)))


## OUTPUT:
![Screenshot (83)](https://user-images.githubusercontent.com/75234807/166142539-ff07f0b7-18e3-493c-901b-9a8e24cfc757.png)

## SOLUTION JUSTIFICATION:
This program gives the solution with minimum stops between locations using breadth first search.

## RESULT:
Thus an algorithm to find the route from the source to the destination point using breadth-first search is developed and executed successfully.
