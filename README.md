# ProjectD
Flood Risk Mapping

This project aims to create a flood risk mapping tool that simulates potential flood zones based on terrain data. The project will utilize Digital Elevation Models (DEMs) to analyze terrain elevation and then simulate flood scenarios under various sea-levels. The end goal is to identify regions with high flood risk and visualize potential flood zones.
Flooding is a critical natural hazard that affects millions of people globally. Identifying flood-prone areas is essential for disaster preparedness, urban planning, and environmental management. This project will leverage Python and publicly available geographic data(AHN) to simulate flood events, providing a valuable tool for assessing flood risk in different regions.

I could not figure out how to link my python code with github. Sorry!

import rasterio
from rasterio.merge import merge
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.widgets import Slider

# File paths for the four elevation tiles
file_paths = [
    '/Users/dimiv/Downloads/R_25GN1.tif',
    '/Users/dimiv/Downloads/R_25DN2.tif',
    '/Users/dimiv/Downloads/R_25EZ1.tif',
    '/Users/dimiv/Downloads/R_25BZ2.tif'
]

# Open the datasets
datasets = []
for fp in file_paths:
    try:
        src = rasterio.open(fp)
        datasets.append(src)
        print(f"Opened file: {fp}, CRS: {src.crs}, Nodata: {src.nodata}")
    except Exception as e:
        print(f"Error opening file {fp}: {e}")

# Merge the datasets
if datasets:
    merged_data, merged_transform = merge(datasets, nodata=np.nan)

    # Convert to float32 and replace nodata values with NaN
    
   merged_data = merged_data[0].astype('float32')  # assuming single-band rasters
    merged_data[merged_data == datasets[0].nodata] = np.nan
    print("Merge successful")
else:
    print("Failed to open one or more files.")

# Set up the plot
fig, ax = plt.subplots(figsize=(10, 8))
plt.subplots_adjust(bottom=0.25)

# Display the merged elevation map with the adjusted scale from -3 to 3
elevation_img = ax.imshow(merged_data, cmap='YlGnBu', aspect='equal', vmin=-3, vmax=3)  # Adjusted color and range
cbar = fig.colorbar(elevation_img, ax=ax, label="Elevation (m)")

# Create a slider for sea level with a range from -3 to 3
ax_slider = plt.axes([0.25, 0.1, 0.65, 0.03])
sea_level_slider = Slider(ax_slider, 'Sea Level', -3, 3, valinit=0)  # Range adjusted for sea level

# Create the initial flood mask (with sea level = 0)
flooded = np.ma.masked_greater(merged_data, sea_level_slider.val)  # Mask everything above sea level
overlay_flood = ax.imshow(flooded, cmap='Reds', alpha=0.7)  # Flood areas in bright red

# Update function for the flood overlay based on sea level
def update(val):
    sea_level = sea_level_slider.val  # Get current sea level from the slider
    flooded = np.ma.masked_greater(merged_data, sea_level)  # Reapply the mask based on sea level
    overlay_flood.set_data(flooded)  # Update the flood overlay with the new masked data
    fig.canvas.draw_idle()  # Redraw the figure to reflect the changes

# Connect the slider to the update function
sea_level_slider.on_changed(update)

# Set title, remove axis labels and ticks
plt.title("Amsterdam Elevation Map with Adjustable Sea Level")
ax.set_xticks([])  # Remove x-axis ticks
ax.set_yticks([])  # Remove y-axis ticks
ax.set_xticklabels([])  # Remove x-axis labels
ax.set_yticklabels([])  # Remove y-axis labels

# Show the plot
plt.show()

