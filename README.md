# NgxMazes

The library is designed to help angular developers create perfect* mazes easily with a graph as scaffolding for movement within the maze.

This library was generated with [Angular CLI](https://github.com/angular/angular-cli) version 13.2.0.

https://user-images.githubusercontent.com/763952/162148955-9810b076-61ac-4e22-a61f-315b6c8965f7.mp4

A maze generated at random with Prims Algorithm. It has had its wall thickened and then is reflected both in the X and Y direction. A central room is added at the end. A pacman maze! Note that canvas animation is not yet a part of this package, which only creates a raw maze data file.

## Todo 
- [ ] Add weave mazes
- [ ] Add additional algorithms (non binary tree, for fun)
- [ ] Add option to auto open the rooms to all adjacent corridors
- [ ] Add output function for SVG
- [ ] Add output function for Canvas
- [ ] Add output function for HTML

## Links

A demonstration version of this will be readily available in the coming days at [My Programming Site](https://physics.sweeto.co.uk/mazes/).

A github link to the public repository can be found [Here](https://github.com/Bogomip/ngx-mazes);


## Add to Project

To add this to your Angular project is simple. From the terminal in your project run `npm install ngx-mazes`, and then in the component you want to use it create an object by `const maze: MazeAlgorithms = new PrimsMaze()` where PrimsMaze may be the name of any algorithm generation method. 

To the make the maze you would use `maze.generateMaze(width of maze, height of maze, optional iterations per second);`. If you input a value other than 0 for the iterations per second (the default) then the maze generator will perform each iteration and then return a maze object via the observable `currentData` to which you must subscribe to get the maze output. The most basic function you might use to get a maze then being:

    const maze: MazeAlgorithms = new PrimsMaze();
    maze.currentData.subscribe((data: IterationData) => { // data.maze is your output, so do something with it. })
    maze.generateMaze(width,height);


## Maze Types

##### Prims Algorithm
Fast and potentially infinite maze generation technique. Use `let maze = new PrimsMaze();`

##### Aldous Broder Algorithm
Slow maze generation technique but can be fun to watch. Very inefficient. Use `let maze = new AldousBroderMaze();`


## Extra functionality and functions

##### Y Mirror 
`mirrorMazeYDirection(Maze2D object, number of wall openings)`

Mirrors the maze in the y direction

##### X Mirror 
`mirrorMazeXDirection(Maze2D object, number of wall openings)`

Mirrors the maze in the x direction

##### Thicken Walls
`makeWallsThick(Maze2D object)`

The mazes generated by default have only walls between passeges which are essentially infinitely thin and can be represented as you wish. However if you want actual, 1 tile wide walls between your passages this function will do that by expanding the maze and setting wall = true to the inbetween bits.

##### Add a Room
`addRoom(Maze2D object, width of room, height of room, x center of room, y center of room)`

Creates a rectangular room of width and height at center point with grid coordinates x, y.

##### Pause generation (for mazes generated over time)
`pauseMaze()`, `playMaze()`

Call these functions to pause maze generation or resume maze generation respectively.

##### Generate a Graph
`generateMazeGraph(maze: Maze2D)`

Generate a graph for the maze. This might be used for an algorithm to create a path through the maze, such as the A* or Dijkstra algorithms.


## Types

Important data types are:

##### Maze2D
`Type Maze2D: { width: number, height: number, tiles: Tile[][] }`

A maze object. 

##### Tile
`Type Tile: { id: string, passable: {l: boolean,r: boolean,t: boolean,b: boolean}; speed: number; wall: boolean; }`

Each tile on the maze is represented as this type, with a wall on the top, left, right and bottom and the possibilitity of being a wall. Each tile has a unique ID and a speed value which is usually set to 1 (might be used for pathfinding algorithms if you wish).

##### IterationData
`IterationData: { maze: Maze2D; i: number; o: number; iteration: number, timeTaken: number, finalIteration: boolean }`

Every iteration returns an IterationData data type, so subscribing to `currentData` will return this after either each iteration or after the final iteration only. The specific parts of this data type are:
- `i` - the last i value, or row, that was iterated on.
- `o` - the last o value, or column, that was iterated on.
- `iteration` - the iteration number, returns 1 for mazes generated immediately.
- `timeTaken` - the time taken since the last iteration, so returns the total time for immediate mazes or the time taken for 1 iteration for mazes generated over time.
- `finalIteration` - if this is the final iteration to expect this returns true.

##### MazeGraph
`MazeGraph: { nodes: MazeNode[] }`

A data type which forms a graph for the maze.

##### MazeNode
`Type MazeNode:{ x: number; y: number; id: string; connections: string[] }`

One node on the graph. Each has a unique string ID and maintains a list of connections to other nodes by storing the connection IDs.


## Example Code

Creating a maze is fairly straightforward and certain helper functions are included to allow easily changes to the mazes.

The following code might be used to generate the maze depicted at the top of the page.

    // start by creating a new maze object. This is what will provide the functionality for the maze.
    let maze: MazeAlgorithms = new AldousBroderMaze();
    let mazeData: IterationData;

    const widthOfMaze: number = 20;
    const heightOfMaze: number = 10;

    // all data which is poured out of the algorithms is done so via an observable. This is because mazes can either be built over time (to show the maze being generated) or instantly.
    this.mazedata = maze.currentData.subscribe((data: IterationData) => {
        
        // get the actual maze structure from data.maze
        maze = data.maze;

        // this next step will create a mirror of the maze in the Y direction with one opening between the two halves.
        maze = maze.mirrorMazeYDirection(maze, 1);

        // this next step will create a mirror of the maze in the X direction with two openings between the two halves.
        maze = maze.mirrorMazeXDirection(maze, 1);

        // this next function makes the walls thicker, so it looks a bit cooler or provides no sharp turns (for games maybe)
        maze = maze.makeWallsThick(maze);

        // the final function adds a room in the middle of the maze, this can also add rooms at any point around the maze. No doors are provided and need to be manually removed for now.
        maze = maze.addRoom(maze);

        // finally generate a graph of the maze
        if(data.finalIteration) {
          this.mazeGraph = maze.generateMazeGraph(maze);
        }
    })

    // generate the maze object
    maze.generateMaze(widthOfMaze, heightOfMaze, 10);
