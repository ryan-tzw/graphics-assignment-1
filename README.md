# OpenGL Mesh Viewer
50.017 Graphics and Visualisation - Assignment 1

# Explanations for each part
## Mesh Loading (LoadInput function)
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