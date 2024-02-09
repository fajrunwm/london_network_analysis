## Import Libraries


```python
# import libraries
import numpy as np
import pandas as pd
import matplotlib
import matplotlib.pyplot as plt
import osmnx as ox        # this line imports osmnx
import networkx as nx     # this line imports networkx
import matplotlib.cm as cm
import matplotlib.colors as colors
import contextily as ctx
```

## Defining study area


```python
# defining networks of UCL with radius of 2000 metres
G=ox.graph_from_address('UCL, London', dist=500, network_type='walk')
#ox.plot_graph(G)
```

## Get OpenStreetMap Geometries

### Cafe


```python
# Define the tags for filtering amenities (in this case, cafes)
tags = {'amenity': 'cafe'}

cafe_geom = ox.features.features_from_address('UCL, London', tags=tags, dist=500)
cafe_geom = cafe_geom.to_crs(epsg=3857)
```


```python
# Plot the cafes
fig, ax = plt.subplots(figsize=(8, 8))
cafe_geom.plot(ax=ax, color='blue', markersize=20, alpha=0.7, label='Cafes')

# Add basemap from CartoDB.Positron
ctx.add_basemap(ax, source=ctx.providers.CartoDB.Positron)

ax.axis('off')
plt.legend()
plt.show()
```

    C:\Users\fajrunwm\AppData\Local\Temp\ipykernel_15700\563875412.py:9: UserWarning: Legend does not support handles for PatchCollection instances.
    See: https://matplotlib.org/stable/tutorials/intermediate/legend_guide.html#implementing-a-custom-legend-handler
      plt.legend()
    


    
![png](bloomsbury_network_analysis_files/bloomsbury_network_analysis_7_1.png)
    


## Network Analysis

### Closeness Centrality


```python
# some of the centrality measures are not implemented on multiGraph so first set as diGraph
DG = ox.get_digraph(G)
```


```python
# calculate its edge closeness centrality: convert graph into a line graph so edges become nodes and vice versa
edge_cc = nx.closeness_centrality(nx.line_graph(DG))

# after this it need to first set the attributes back to its edge
nx.set_edge_attributes(DG, edge_cc,'cc')

# and turn back to multiGraph for plotting
G = nx.MultiGraph(DG)

# convert graph to geopandas dataframe
gdf_edges = ox.graph_to_gdfs(G,nodes=False,fill_edge_geometry=True)

# set crs to 3857 (needed for contextily)
gdf_edges = gdf_edges.to_crs(epsg=3857) # setting crs to 3857

# Plot the cafes and edges
fig, ax = plt.subplots(figsize=(8, 8))

# Plot the cafe geometries
cafe_geom.plot(ax=ax, color='red', markersize=20, label='Cafes')

# Plot the edges according to closeness centrality
gdf_edges.plot(ax=ax, column='cc', cmap='plasma', label='edges', linewidth=1, legend=True, legend_kwds={'shrink': 0.8})

# Add basemap from CartoDB.Positron
ctx.add_basemap(ax, source=ctx.providers.CartoDB.Positron)

ax.axis('off')
plt.title('Map of cafe with closeness centrality index')
plt.legend()
plt.show()
```

    C:\Users\fajrunwm\AppData\Local\Temp\ipykernel_15700\1196762778.py:30: UserWarning: Legend does not support handles for PatchCollection instances.
    See: https://matplotlib.org/stable/tutorials/intermediate/legend_guide.html#implementing-a-custom-legend-handler
      plt.legend()
    


    
![png](bloomsbury_network_analysis_files/bloomsbury_network_analysis_11_1.png)
    


### Betweenness Centrality


```python
# similarly, let's calculate edge betweenness centrality: convert graph to a line graph so edges become nodes and vice versa
edge_bc = nx.betweenness_centrality(nx.line_graph(DG))

# after this it need to first set the attributes back to its edge
nx.set_edge_attributes(DG,edge_bc,'bc')

# and turn back to multiGraph for plotting
G = nx.MultiGraph(DG)

# convert graph to geopandas dataframe
gdf_edges = ox.graph_to_gdfs(G,nodes=False,fill_edge_geometry=True)

# set crs to 3857 (needed for contextily)
gdf_edges = gdf_edges.to_crs(epsg=3857) # setting crs to 3857

# Plot the cafes and edges
fig, ax = plt.subplots(figsize=(8, 8))

# Plot the cafe geometries
cafe_geom.plot(ax=ax, color='red', markersize=20, label='Cafes')

# Plot the edges according to betweenness centrality
gdf_edges.plot(ax=ax, column='bc', cmap='plasma', label='edges', linewidth=1, legend=True, legend_kwds={'shrink': 0.8})

# Add basemap from CartoDB.Positron
ctx.add_basemap(ax, source=ctx.providers.CartoDB.Positron)

ax.axis('off')
plt.title('Map of cafe with betweenness centrality index')
plt.legend()
plt.show()
```

    C:\Users\fajrunwm\AppData\Local\Temp\ipykernel_15700\1470670966.py:30: UserWarning: Legend does not support handles for PatchCollection instances.
    See: https://matplotlib.org/stable/tutorials/intermediate/legend_guide.html#implementing-a-custom-legend-handler
      plt.legend()
    


    
![png](bloomsbury_network_analysis_files/bloomsbury_network_analysis_13_1.png)
    


### Shortest Path


```python
# shortest path
origin_point = ox.geocode('Oxford Circus, London')
destination_point = ox.geocode('Euston Station, London')

origin_node = ox.nearest_nodes(G, origin_point[1], origin_point[0])
destination_node = ox.nearest_nodes(G, destination_point[1], destination_point[0])
origin_node, destination_node

route = nx.shortest_path(G, origin_node, destination_node, weight='length')
str(route)

fig,ax = ox.plot_graph_route(G, route)
```


    
![png](bloomsbury_network_analysis_files/bloomsbury_network_analysis_15_0.png)
    

