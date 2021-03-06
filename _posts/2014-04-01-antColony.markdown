---
layout: post
title:  "Ant Colony"
date:   2016-01-01
categories: posts
author: "Ayoub"
tag : "Building"
description: "Basic Ant Colony algorithm"
---

In this post I'll present a basic Ant Colony Algorithm allowing the visualization of the ants behavior. It is far from being final but it can be easily tweaked and modified. The best part: it uses PyGame to visualize the ants moving scattering the pheromone. The code can be found on the <a href="https://github.com/ayourriss/AntColony"><span style="font-size: 14pt;"><strong><span style="font-size: 12pt;"><span style="font-size: 10pt;"><span style="color: #0000ff;">Github repository</span>.</span></span></strong></span></a> If you encounter any problems with the code, please rise an issue on Github.

Th ant behavior have inspired researchers to build a class of algorithms called "genetic algorithms". In a genetic algorithm, a population of candidate solutions (ants in our case) to an optimization problem is evolved toward better solutions. Once an ant finds the food location, it starts scattering a higher quantity of pheromone in its neighbor so other ants can detect the food location. The future "generation", based on these pheromone traces, have better chances of finding a shorter path. At this stage, the decision parameters are constant and common to all the colony ants [latex] ( \alpha , \beta) [/latex], therefore my algorithm doesn't integrate population mutation, the current changes affect only the environment through the pheromone traces.

In the following sections, I will be describing the code functions and what each class does/doesn't.

Launch main.py to run the colony. Running the code requires the following packages : pygame, numpy, random, bisect. The algorithm parameters are stored in params.py.
<h2 style="padding-left: 30px;"><strong>I- The Grid:</strong></h2>
The ants move in a Grid object (defined in <span style="font-family: courier new,courier,monospace; font-size: 10pt;">grid.py</span>). The grid is a matrix of Cell objects of size : <span style="font-family: courier new,courier,monospace; font-size: 10pt;">grid_size</span>, used to reiterate operations across all cells.

[caption id="attachment_76" align="aligncenter" width="734"]<img class="wp-image-76" src="http://data.ghriss.net/ant/ant1.jpg" alt="Ant Colony" width="734" height="357" /> Ant Colony - Main Screen[/caption]
<h3 style="padding-left: 30px;"><strong>1-1 The Cell class:</strong></h3>
The Cell class has the following attributes:
<ul>
 	<li>Coordinates : [latex](x,y)[/latex]</li>
 	<li>Number of ants in the cell : [latex]count[/latex]</li>
 	<li>Quantity of pheromone in the current cell : [latex]phero[/latex]</li>
 	<li>Pheromone lower and upper bound: [latex]phero_{min},phero_{max}[/latex]</li>
 	<li>Evaporation rate : [latex]evaporate[/latex]</li>
 	<li>Cell type : [latex] \in (Wall, Road, Nest, Food) [/latex]</li>
</ul>
<p style="padding-left: 10px;">The Cell has 2 methods :</p>

<ul>
 	<li>Update : evaporate the pheromone</li>
 	<li>Draw : given the pygame display, draw the cell color (reflects the cell type, defined in Colors dictionary in cell.py)</li>
</ul>
<h3 style="padding-left: 30px;"><strong>1-2 The grid controls :</strong></h3>
The grid can be either drawn in the interface or loaded from the file "map.txt", in which case, each line has the format : int i,int j,string s. With i,j : coordinates of the cell and s the type of the cell.

Once the first main screen in displayed, press "ESC" to start editing the grid. Use left mouse button to draw wall and right button to delete a wall.
<h2 style="padding-left: 30px;"><strong>II- Ant Colony :</strong></h2>
<img class="size-medium wp-image-181 aligncenter" src="https://data.ghriss.net/ant/ant.gif" alt="Ant Colony Working" width="291" height="300" />

The Colony object (colony.py) is a list of Ants, with one method "work" calling the work method of all ants. It can be used later to assign different movement parameters to ants. The idea is to attribute random parameters for the first generations. As more ants are being generated, the algorithm will try to center next generation's parameters around those of ants who reached the nest carrying food.

Once an Ant (ant.py) has been initialized, the method 'work' represents the ant brain. The ant follows the next algorithm:
<h4 style="padding-left: 30px;"><strong>Step 1 - Start from a random nest with no food (__init__):</strong></h4>
<p style="padding-left: 30px;">The ant needs to know :</p>

<ul style="padding-left: 60px;">
 	<li>The grid it will be moving in (grid)</li>
 	<li>Its movement parameters :
<ul>
 	<li>Directions weights : [latex](\alpha,\beta)[/latex]</li>
 	<li>Pheromone scattering :[latex]phero_{max},phero_{min}[/latex]</li>
 	<li>Maximum movement range : distance</li>
</ul>
</li>
</ul>
<h4 style="padding-left: 30px; column-width: 80%;"><strong>Step 2 - Explore the grid (work):</strong></h4>
<p style="padding-left: 60px;"><strong>Step 2-1: </strong>Choose next move's direction (choose_direction).</p>
The directions are 2D vectors [latex](x_i,y_i)[/latex] with [latex]x_i = 1[/latex] for East, [latex]-1[/latex] for West, 0 otherwise,  and [latex]y_i = +1[/latex] for North, [latex]-1[/latex] for South, 0 otherwise. To ease the operations, I used a maping (dictonary Directions) [latex] {0,1...7} \to {N, NE ..... NW}[/latex], in clockwise rotation, starting from 0 as North to 7 as North West. The inverse mapping is the dictionary Labels.

Using the labels, if an Ant has direction [latex]j[/latex], then 180 degree rotation is simply [latex] i_{back} = (j+4)\% 8[/latex] and a right-turn label is [latex] i_{right} = (j +2)\% 8[/latex]  (where [latex]\%x[/latex] the reminder of euclidean division by x ). Using the introduced notations, the weight of the direction j is :
<p style="text-align: center;">[latex] w_j^* = Pheromone(\text{current position} + direction_j)^\alpha.||direction_j||_2^\beta[/latex]</p>
[latex]Pheromone(x,y)[/latex] is the pheromone quantity in the Cell(x,y). With one step per iteration, [latex]||direction_j||_2 = \sqrt{2}[/latex] when moving diagonally (odd labels),1 otherwise (even labels).

To make the ants move forward, we can put more weights on keeping the same direction. I have in particular tried the weights [latex]\hat{w}_j = \frac{w_j^*}{|i-j|+1} [/latex], where [latex]i [/latex] is the current direction's label.
<p style="padding-left: 60px;"><strong>Step 2-2:</strong> If the next direction is accessible: scatter pheromone (scatter_phero) and move with one step. If not: turn right (rotate).</p>
<p style="padding-left: 90px;">The pheromone scattering call 'when_has_food' and 'when_no_food' specifying the pheromone quantity to scatter for each case.</p>
<p style="padding-left: 60px;"><strong>Step 2-3:</strong> If current location :</p>
<p style="padding-left: 90px;">i) has food then mark ant as "has food", turn back and restart from step 2-1.</p>
<p style="padding-left: 90px;">ii) If in the nest then reset the ant, otherwise, restart from 2-1.</p>

<h2>III - The application:</h2>
The App class is a Pygame interface that displays the grid and allows walls modification. It has no particular algorithm: it creates a Pygame display of a resolution = block_size * grid_size, then, at each iteration:
<ul>
 	<li>Call the method 'work' of Colony</li>
 	<li> Update the Grid</li>
 	<li>Draw the Grid</li>
 	<li>Refresh the display</li>
</ul>
I hope you enjoy playing with the ants. You can toy with parameters [latex]\alpha,\beta[/latex], for example: taking a very negative [latex]\beta[/latex] will make ants move in straight lines (army marching).

If you encounter a problem when running the code then please raise the issue in the Github repository.

&nbsp;
