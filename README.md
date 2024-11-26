# USAPI-Mapping-Tutorial

<div style="display: flex; justify-content: center; align-items: center; gap: 20px; margin: 20px 0;">
  <img src="https://github.com/user-attachments/assets/a6f0c893-e9fc-412a-ba9b-4cf4b9049b25" alt="CIFAL Logo" style="width: 200px; height: auto;">
  <img src="https://github.com/user-attachments/assets/9803cc72-85fe-4cd7-a956-2e3c4f528596" alt="Spice Logo" style="width: 200px; height: auto;">
</div>

### Student Contributors

[Subin Cho](https://github.com/SubinCho26)

Sarah Carrol

### Overview

A significant amount of the data we absorb is through GIS/spatial visualization. Due to geographic location and size, the **USAPI** is invisible in standard choropleth maps of the US.

From [PIHOA](https://www.pihoa.org/usapi-region/), The USAPI include the three U.S. Flag Territories of Guam, the Commonwealth of the Northern Mariana Islands, and American Samoa, as well as the three Freely Associated States (independent nations in a special compact relationship with the United States) of the Republic of Palau, the Republic of the Marshall Islands, and the Federated States of Micronesian (Pohnpei, Kosrae, Chuuk, and Yap).

The USAPI are populated by more than 500,000 inhabitants who live on hundreds of remote islands and atolls spanning millions of square miles of the Pacific Ocean and crossing five time zones, including the international dateline.  These islands are culturally and linguistically diverse with more than a dozen spoken languages. While the indigenous peoples of the USAPI are rich in culture they are considerably small in population.  The islands are socially, politically and economically fragile but they are bountiful with rich marine and land-based eco-systems and numerous wildlife that cannot be found anywhere else on earth.

Despite the distance and isolation between islands, multiple complex factors contribute to the severe health disparities and outcomes. The current health infrastructure in the USAPI suffers from severe resource limitations. Health status indicators demonstrate significant disparities from across the Public Health spectrum. Factors influencing policy issues, political relationships, the economy, the environment, diverse cultures, stressed health systems, education, limited human resource development and the sheer physical isolation of these islands all contribute to the enormous challenges in achieving health equity. Colonization and rapid westernization have adversely affected many of the social, cultural, and environmental structures and practices that traditionally supported and protected the health of the islands, their waters and their people. 

### Objective

Map the USAPI in a way where it can be compared to US States via choropleth mapping.

-   Limitations

    -   This design is practical for many visualizations but can cause:

        -   **Misinterpretation of actual locations** if the map isn't labeled correctly.

        -   A **loss of spatial context**, especially if someone unfamiliar with U.S. territories is viewing the map.

### Tools

<img src="https://github.com/user-attachments/assets/6dd0fbad-5bc4-4881-99de-fc5c5383d089" alt="Image" style="width: 100px; height: auto;">


The [urbnmapr](https://github.com/UrbanInstitute/urbnmapr) package in R allow us to perform this mapping simply with the `get_urbn_map()` function.

### Tutorial

#### Load necessary packages

```{r, message=FALSE}
library(tidyverse)
library(urbnmapr)
library(here)
library(ggiraph)
library(sf)
```

#### Load in the Data

```{r, message=FALSE}
territories_states <- get_urbn_map(map = "territories_states", sf = TRUE)
```

#### Enlarge the territories

```{r}
# Step 2: Identify and filter territories to be enlarged
territories_to_expand <- territories_states %>%
  filter(state_abbv %in% c("GU", "MP", "AS"))  # GU = Guam, MP = Mariana Islands, AS = American Samoa

# Step 3: Center and scale geometries (keep positions fixed)
territories_to_expand_scaled <- territories_to_expand %>%
  mutate(
    # Calculate centroids of each geometry
    centroid = st_centroid(geometry),
    # Center geometries around (0, 0) for scaling
    geometry_centered = st_geometry(.) - st_geometry(centroid),
    # Apply scaling factor
    geometry_scaled = geometry_centered * 2,  # Adjust the factor as needed
    # Re-center geometries to their original positions
    geometry = geometry_scaled + st_geometry(centroid)
  ) %>%
  select(-centroid, -geometry_centered, -geometry_scaled)  # Clean up intermediate columns

# Step 4: Ensure CRS Consistency
# Assign CRS to scaled geometries if missing
territories_to_expand_scaled <- st_set_crs(
  territories_to_expand_scaled, 
  st_crs(territories_states)  # Use CRS from the original dataset
)

# Align CRS of scaled geometries with the original dataset
territories_to_expand_scaled <- st_transform(
  territories_to_expand_scaled, 
  crs = st_crs(territories_states)
)

# Combine scaled territories with the original map
territories_adjusted <- territories_states %>%
  filter(!state_abbv %in% c("GU", "MP", "AS")) %>%  # Exclude the original territories
  bind_rows(territories_to_expand_scaled)
```

#### Add interactive layers

```{r}
territories_adjusted <- territories_adjusted %>%
  mutate(tooltip_info = paste("State/Territory:", state_name, "<br>Abb:", state_abbv))
```

#### Plot

```{r}
# Step 5: Create the interactive map
gg <- ggplot() +
  geom_sf_interactive(
    data = territories_adjusted,
    aes(geometry = geometry, tooltip = tooltip_info, data_id = state_abbv), # data_id for hover effects
    fill = "white", color = "black"
  ) +
  theme_minimal(base_size = 10) +
  theme(
    plot.background = element_rect(fill = "black", color = "black"),  # Black background
    panel.background = element_rect(fill = "black", color = "black"),
    legend.background = element_rect(fill = "black"),
    text = element_text(color = "steelblue"),  # White text
    panel.grid = element_blank(),
  axis.text = element_blank(),         # Remove axis text (latitude and longitude)
  axis.ticks = element_blank(),        # Remove axis ticks
  axis.title = element_blank(),
  plot.margin = margin(t = 8, r = 10, b = 75, l = 10) # Remove axis titles
) +
  labs(title = "US & USAPI for Chloropleth Mapping",
       caption = "Hover to see details.")

# Render the interactive map with custom hover effects
girafe(
  ggobj = gg, 
  options = list(
    opts_hover(css = "fill: steelblue; stroke: yellow; stroke-width: 2px;"),  # Highlight on hover
    opts_tooltip(css = "background-color: white; color: black; border-radius: 5px; padding: 5px;")
  )
)
```
<img width="763" alt="Screen Shot 2024-11-25 at 4 15 55 PM" src="https://github.com/user-attachments/assets/44fbd0f4-c5ac-4d36-a0fe-0d7a92d38a0f">



