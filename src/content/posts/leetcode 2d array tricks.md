---
  title: leetcode 2d array tricks
  pubDate: 2024-08-08
  categories: []
  description: ''
---

Every single time I deal with a 2D array in leetcode, I always end
up writing/copying the same exact code to
make the code simpler to write 
with less cognitive overhead.

First, we have the `Location` class. Instead of 
everything being a tuple and I have to constantly figure
out what's the x and y, I use this.

```python
@dataclass(frozen=True, eq=True, unsafe_hash=True)
class Location:
    x: int = -1
    y: int = -1


    def __add__(self, other: "Location") -> "Location":
        return Location(self.x + other.x, self.y + other.y)


    def __mul__(self, other: int) -> "Location":
        return Location(self.x * other, self.y * other)
```

We have the add and multiply operator overloaded so we can easily navigate
the 2D array.

Then I always end up needing at least the following constants.

```python
UP = Location(0, -1)
DOWN = Location(0, 1)
LEFT = Location(-1, 0)
RIGHT = Location(1, 0)

DIRECTIONS = [UP, DOWN, LEFT, RIGHT]
```

This allows for code like this.

```python
for direction in DIRECTIONS:
    new_location = current_location + direction
```

Then I always need to create a new class for 
the 2D array. For two reasons, 
one is that negative indexing is evil. I've had so many bugs because I accidentally
wrapped around the array.

The second reason is that I want to easily index the 2D array with a `Location` object.

```python
T = TypeVar("T")
@dataclass
class Board(Generic[T]):
    grid: list[list[T]]

    def __getitem__(self, location: Location) -> T:
        assert 0 <= location.y < len(self.grid), f"Y out of bounds: {location.y}"
        assert 0 <= location.x < len(self.grid[location.y]), f"X out of bounds: {location.x}"
        return self.grid[location.y][location.x]

    def __setitem__(self, location: Location, value: T) -> None:
        assert 0 <= location.x < len(self.grid[0]), f"X out of bounds: {location.x}"
        assert 0 <= location.y < len(self.grid), f"Y out of bounds: {location.y}"
        self.grid[location.y][location.x] = value

    def __iter__(self) -> Tuple[Location, T]:
        for y, row in enumerate(self.grid):
            for x, value in enumerate(row):
                yield Location(x, y), value

```
This allowed me to do this leetcode problem in the following way.
https://leetcode.com/problems/shortest-bridge/description/

```python
WATER = 0
ISLAND = 1
ON_FIRST_ISLAND = 2
WATER_CHECKED = 3

class Solution:
    def shortestBridge(self, grid: List[List[int]]) -> int:
        # find a location that is part of any island
        board = Board(grid)
        first_one = Location()

        for location, value in board:
            if value == ISLAND:
                first_one = location
                break

        board[first_one] = ON_FIRST_ISLAND
        
        # Do a DFS to find all islands that are connected to the previous island
        queue = [first_one]
        first_island = {first_one}

        while queue:
            location = queue.pop(0)
            for direction in DIRECTIONS:
                new_location = location + direction
                try:
                    if board[new_location] == ISLAND:
                        board[new_location] = ON_FIRST_ISLAND
                        queue.append(new_location)
                        first_island.add(new_location)
                except AssertionError:
                    pass



        # Do a BFS to find the shortest path from the first island to any other island
        queue = list(first_island)
        distance = 0

        while queue:
            new_queue = []
            for location in queue:
                for direction in DIRECTIONS:
                    new_location = location + direction
                    try:
                        if board[new_location] == WATER:
                            board[new_location] = WATER_CHECKED
                            new_queue.append(new_location)
                        elif board[new_location] == ISLAND:
                            return distance
                    except AssertionError:
                        pass
            queue = new_queue
            distance += 1

        return -1
```

