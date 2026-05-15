---
trigger: always_on
description: Cursor rules for WebAssembly development with Z80 Cellular Automata integration.
---

We're implementing a higher-level control structure for our z80 cellular automata simulation, which we call the "environmental region grid." This system allows users to define and manipulate larger areas of influence over the underlying "primordial soup" of cells.

Key Concepts:

1. Soup Cells: The individual units of our cellular automata, which follow basic rules and interact with their neighbors.
2. Regions: Larger areas that encompass multiple soup cells. Each region can have unique properties that influence the behavior of the soup cells within it.
3. Environmental Region Grid: A grid overlaid on top of the soup cell grid, dividing the simulation space into discrete regions. This grid can be 4x4, 8x8, or 16x16, allowing for different levels of granularity.
4. Region Parameters: Each region has a set of adjustable parameters that affect the soup cells within it. These could include:
   - Obstacle (A region that blocks the movement of soup cells)
   - Directional influence (biasing cell interactions in specific directions)
   - Randomness factor (introducing more or less chaos in cell behavior)
   - Temperature (affecting overall activity levels)
   - Energy levels (influencing the likelihood of certain cell states or interactions)
   - Other custom parameters as needed
5. Dynamic Influence: The region parameters dynamically modify the behavior of soup cells, creating areas of distinct characteristics within the larger simulation.
6. User Interaction: Users can interact with the simulation by adjusting region parameters in real-time, allowing for on-the-fly modification of the simulation's behavior.
7. Visualization: The region grid and its effects are visually represented, allowing users to see the influence of their changes on the simulation.

Purpose:

This system adds a new layer of complexity and control to the cellular automata simulation. It allows for the creation of diverse environments within a single simulation, enabling users to explore how different regional properties affect the emergent behavior of the cellular automata.

By implementing this region grid system, we're providing a powerful tool for users to experiment with large-scale influences on cellular automata behavior, potentially leading to new insights and interesting emergent phenomena.

Plan:

1. Define the Region Structure:
   Create a comprehensive data structure to represent each region. This structure should be flexible enough to accommodate various parameters that can influence the behavior of soup cells within that region. Consider including:
   - Obstacle
   - Directional influence (for each cardinal direction)
   - Randomness factor
   - Temperature
   - Energy level
   - Any other relevant parameters
   Ensure that each parameter is represented by an appropriate data type, typically using floating-point numbers for continuous values or integers for discrete states. This structure will be the foundation of your region system, so design it with extensibility in mind.

2. Create the Region Grid:
   Implement a two-dimensional array to represent the region grid. This grid should be flexible in size, allowing for configurations such as 4x4, 8x8, or 16x16. Each element of this array will be an instance of the region structure defined in step 1. Initialize this grid with default values for all parameters, ensuring a consistent starting state. Consider implementing methods to easily resize the grid and maintain the aspect ratio with the underlying soup cells.

3. Implement Soup Cell to Region Mapping:
   Develop a system to efficiently map each soup cell to its corresponding region. This mapping is crucial for quick lookups during simulation. Create a separate array where each element represents a soup cell and contains the index or reference to its associated region. Implement functions to update this mapping whenever the region grid size changes. Ensure that this mapping system is optimized for performance, as it will be frequently accessed during the simulation.

4. Modify the Main Simulation Loop:
   Update the core simulation logic to incorporate region parameters. For each soup cell update:
   a. Determine the cell's corresponding region using the mapping created in step 3.
   b. Retrieve the region's parameters.
   c. Apply the effects of each parameter to the soup cell's behavior.
   This might involve adjusting probabilities, modifying state transition rules, or influencing the cell's interaction with neighbors. Ensure that this integration is done efficiently to maintain simulation performance.

5. Implement Parameter-Specific Logic:
   For each parameter in the region structure, create dedicated functions or methods to apply its effects. For example:
   - Obstacle: Turns the cell into an obstacle, preventing it from being randomly selected, and preventing neighbor soup cells from interacting with it.
   - Directional influence: Adjust the probability of a cell interacting with neighbors in specific directions.
   - Randomness: Introduce variability in state transitions or cell behavior.
   - Temperature: Affect the overall activity level or energy of cells within the region.
   - Energy level: Influence the likelihood of certain operations or state changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
