# OpenGL Mesh Viewer
50.017 Graphics and Visualisation - Assignment 1

# Explanations for each part
## LoadInput (mesh loading)
- `std::ifstream` is used to load the OBJ file first
- Then we read the file line by line
- If it's a vertex or normal (line starts with v or vn), we read the xyz values, and store them in a vector of `glm::vec3`
- We do this so that when we read the faces, we can use the indices to get the correct vertex and normal data
- Once we read a face (line starts with f), we process each vertex in the face
	- We pull out the v, vt (unused) and vn indices
	- Using the v and vn indices, we create a `std::pair<int, int>` to represent each unique vertex/normal pair
		- If a vertex at the same location has multiple normals it gets treated as multiple vertices
		- But it seems like the models we were given don't have this issue
	- Using the pair created previously as a key we check a map to see if this vertex already exists
		- if yes we just push the index
		- if not we create a new vertex, add it to the vertex list, and store the index in the map


## SetMeshColor
```
colorID = (colorID + 1) % 4;
```
- Not much to say, just increments the colorID and wraps around using modulo

## TranslateModel
```
modelMatrix = glm::translate(glm::mat4(1.0f), transVec) * modelMatrix;
```
- We apply the modelMatrix first so that we can work in world space instead of local space
	- Technically if we wanted this to work in the general case (when the camera is anywhere in space), we would need to work in view space instead.
	- But since our camera is always on the Z axis looking at the world origin, this works fine enough
- If we don't apply the modelMatrix first (i.e. we try to work in local space instead), then when trying to move the model "left and right" (along the X axis) 
  it would move along its local X axis, which would ALSO be rotated if the model was rotated already.
- Then we simply translate in world space. `transVec` is a "3D" vector but it only has values in the X and Y components from dX and dY from the mouse input.
	- We provide `glm::translate()` with an identity matrix and our translation vector to get a translation matrix in world space


## RotateModel
```
modelMatrix = glm::rotate(glm::mat4(1.0f), angle, axis) * modelMatrix;
```
- Similar to translation, we apply the modelMatrix first so that we can work in world space instead of local space
- Similar problem as translation, if we didn't apply the modelMatrix first, rotating the model would rotate it around its local axes.
	- e.g. if the model is yawed (rotated around Y) by 90 degrees for example, then trying to pitch it (X-axis) would appear to cause it to roll 
	(Z-axis) instead because the local X axis is now aligned with the world Z axis after being rotated 90 degrees.


## ScaleModel
```
modelMatrix = glm::scale(glm::mat4(1.0f), glm::vec3(scale, scale, scale)) * modelMatrix;
```
- I originally had this as just `modelMatrix = glm::scale(modelMatrix, glm::vec3(scale, scale, scale));` because I thought it didn't matter whether we scaled in local or world space.
- But, I realised there is an advantage to scaling in world space. 
- In local space, if the model had been moved around like to the side, then when you try scaling it down for example, the model will shrink towards its local origin which is off to the side.
- Whereas, if you do the scaling in world space, the model will shrink (or expand) towards the world origin (0,0,0) which is at the center of the screen.
- And since the camera is always looking at the world origin, this gives the impression of the camera "zooming in and out" on the model which feels like a better user experience.