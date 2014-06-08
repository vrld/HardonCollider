#Line Casts in HardonCollider

I was working on some personal projects with HardonCollider, and I came across a situation where I wanted something like a raycast or a linecast.
I knew Hardon used a spatial hash, so after a quick Google search I found this page,

[Ray casting in a Spatial Hash with DDA](http://www.playchilla.com/ray-casting-spatial-hash-dda)

and after about a week of tinkering, I got something working.

#Example

The following Lua file demonstrates the "shapesOnLine" function.

shapesOnLine takes four arguments,
 
x1, y2, x2,and y2, 
 
which are the coordinates of the two endpoints of the line.

Using the DDA algorithm, the function looks at each cell the line passes through and calls intersectRay on each object in each cell. only the cells the line passes through are checked.

The return value of the function is a table containing all the shapes the line passed through. They are not in any guaranteed order, and currently it may be possible to have duplicates in the table.

Run it with love using my fork of Hardon to see it in action.

the changes I made to the library are probably bad, but I'm happy I got it working.
My current goal for this project is to tidy up what's there by moving the DDA algorithm into the spatialHash class. 

````lua
--main.lua
HC = require 'hardoncollider'

function love.load()
  Collider = HC(25)
  
  --default endpoints of the lineCast
  originX = 350
  originY = 350
  endX = 350
  endY = 450
  
  --set of rectagle shapes for the LineCast to hit
  blocks = {}
  for i=0,39,1 do
    blocks[i]={}
    for j=0,29,1 do
      blocks[i][j] = Collider:addRectangle(20*i+5,20*j+5,10,10);
    end 
  end
end

function love.update(dt)
  Collider:update(dt)
  
  --position the endpoints of the line based on mouse position
  if love.mouse.isDown("r") then
    endX = love.mouse.getX()
    endY = love.mouse.getY()
  elseif love.mouse.isDown("l") then
    originX = love.mouse.getX()
    originY = love.mouse.getY()
  end
end

function love.draw()
  --draw outline of all blocks
  for _,table in pairs(blocks) do
    for _,object in pairs(table)do
      object:draw("line")
    end
  end
  
  --get table of all shapes under the line
  hits = Collider:shapesOnLine(originX,originY,endX,endY,true)
  
  --fill in all the shapes under the line
  for k,v in pairs(hits) do
    v:draw("fill")
  end
  
  --draw tooltips
  love.graphics.setColor(0,100,255,200)
  love.graphics.rectangle("fill",endX,endY,100,20)
  love.graphics.rectangle("fill",originX,originY,100,20)
  love.graphics.setColor(255,0,0,255)
  love.graphics.print(" RMB to move",endX,endY)
  love.graphics.print(" LMB to move",originX,originY)
  
  --draw the line
  love.graphics.line(originX,originY,endX,endY)
  love.graphics.setColor(255,255,255,255)
end
````