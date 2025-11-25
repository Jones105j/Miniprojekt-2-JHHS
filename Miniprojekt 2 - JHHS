import pygame
from perlin_noise import PerlinNoise
from queue import PriorityQueue

# ============================================================
# Global constants - Colors and costs
# ============================================================

BLUE = (23, 99, 199)
GREEN = (81,168,38)
BROWN = (70, 60,50)
RED = (255,0,0)
LIGHT_BLUE = (82, 174, 228)
ORANGE = (255,128,0)
YELLOW = (255,255,0)

TERRAIN_COST = {BLUE: 6, LIGHT_BLUE: 3, GREEN: 1, BROWN: 8}

# ============================================================
# Class: Clicks – To handle mousecliks (start and stop)
# ============================================================
class Clicks:
    def __init__(self):
        #Click related variables
        self.marker_size = 8
        self.marker_color = [ORANGE, RED]
        self.click_positions = []

    def add_click(self, pos):
        #Add click position to list
        self.click_positions.append(pos)
    
    def draw_clicks(self, surface):
        #Draw click positions
        for clicks, (x, y) in enumerate(self.click_positions):
            rect = (x, y, self.marker_size, self.marker_size)
            color = self.marker_color[clicks % len(self.marker_color)]
            pygame.draw.rect(surface, color, rect, 0)

    def clear(self):
        #Clear the click_positions list
        self.click_positions.clear()

# ============================================================
# Class: TerrainMap – Generates and draws perlin-noise terrain
# ============================================================
class TerrainMap:
    def __init__(self,map_size=800,tile_size=8,noise_scale=280,octaves=2,seed = 0):
        #Map related variables
        self.map_size = map_size
        self.tile_size = tile_size
        self.noise_scale = noise_scale
        self.octaves = octaves
        self.seed = seed
        self.noise = PerlinNoise(octaves=self.octaves, seed=self.seed)
        self.tile_color = {}

    def get_color(self, height):
        if height < 0.34:       # Deep water
            return BLUE
        elif height < 0.40:     # Water
            return LIGHT_BLUE
        elif height < 0.56:     # Grass
            return GREEN
        else:                   # Mountain
            return BROWN

    def render(self, surface):
        #Clear previous colors, if a new map i generated
        self.tile_color.clear()
        
        #Draw perlin noise map on surface
        for col in range(0, self.map_size, self.tile_size):
            for row in range(0, self.map_size, self.tile_size):
                
                #Scaling pixel coordinates (stretch noise)
                noise_x = (col / self.noise_scale)
                noise_y = (row / self.noise_scale)
                
                noise_value = self.noise([noise_x, noise_y]) #Get noise value (–1 to 1) for this tile 
                height = (1.0 + noise_value) / 2.0 #Used ChatGPT to change scale from (-1 to 1) to (0 to 1)
                color = self.get_color(height)

                #Draw tiles on screen
                pygame.draw.rect(surface, color, (col, row, self.tile_size, self.tile_size))

                #Pixel coordiantes to grid-cordinates
                grid_x = col // self.tile_size
                grid_y = row // self.tile_size

                #Save color for tile
                self.tile_color[(grid_x, grid_y)] = color


# ============================================================
# Class: GridGraph – Defining grid for pathfinding
# ============================================================
class GridGraph:
    def __init__(self, terrain: TerrainMap):
        self.terrain = terrain
        self.cols = terrain.map_size // terrain.tile_size
        self.rows = terrain.map_size // terrain.tile_size

    def in_bounds(self, grid_pos):
        #Grid position
        col, row = grid_pos

        #Check if column and row is inside the grid
        inside_x = 0 <= col < self.cols
        inside_y = 0 <= row < self.rows

        #Return True if column and row are inside the grid
        return inside_x and inside_y

    def neighbors(self, grid_pos):
        #Current tile in grid
        col, row = grid_pos

        #List of possible neighbor positions
        candidate_neighbors = [
            (col + 1, row), #Right
            (col - 1, row), #Left
            (col, row + 1), #Down
            (col, row - 1), #Up
        ]

        valid_neighbors = []

        #Keep neighbors within the grid boundaries
        for neighbor in candidate_neighbors:
            if self.in_bounds(neighbor):
                valid_neighbors.append(neighbor)

        return valid_neighbors

    def cost(self, from_pos, to_pos):
        #Movement cost
        col, row = to_pos
        color = self.terrain.tile_color[(col,row)]
        return TERRAIN_COST[color]


# ============================================================
# Class: Algorithm – Dijkstras algoritme
# ============================================================
class Algorithm:
    def __init__(self, graph: GridGraph, start_grid_pos, goal_grid_pos):
        self.graph = graph
        self.start = start_grid_pos
        self.goal = goal_grid_pos

    def run_algorithm(self):
        #Priority queue for nodes to explore (always takes lowest cost first)
        frontier = PriorityQueue()
        frontier.put((0, self.start))

        #Track where we came from and current best cost to each node
        came_from = {self.start: None}
        cost_so_far = {self.start: 0}

        #Main Dijkstra loop
        while not frontier.empty():
            current_priority, current = frontier.get()

            #Goal reached == Stop
            if current == self.goal:
                break

            #Check all valid neighbors of the current grid position
            for next in self.graph.neighbors(current):
                new_cost = cost_so_far[current] + self.graph.cost(current, next)

                #If the path to 'next' is better = Update and push to frontier
                if next not in cost_so_far or new_cost < cost_so_far[next]:
                    cost_so_far[next] = new_cost
                    frontier.put((new_cost, next))
                    came_from[next] = current

        return came_from, cost_so_far


# ============================================================
# Main
# ============================================================
def main():
    pygame.init()

    terrain = TerrainMap()
    screen = pygame.display.set_mode((terrain.map_size, terrain.map_size))
    surface = pygame.Surface((terrain.map_size, terrain.map_size))
    clock = pygame.time.Clock()
    graph = GridGraph(terrain)
    clicks = Clicks()
    terrain.render(surface)

    path = []

    while True:
        for event in pygame.event.get():

            #Save left-click position 
            if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                clicks.add_click(event.pos)

                #Grid position for start and goal
                if len(clicks.click_positions) == 2:
                    start_pos, goal_pos = clicks.click_positions
                    start = (start_pos[0] // terrain.tile_size, start_pos[1] // terrain.tile_size)
                    goal  = (goal_pos[0]  // terrain.tile_size, goal_pos[1]  // terrain.tile_size)

                    algorithm = Algorithm(graph, start, goal)
                    came_from, cost_so_far = algorithm.run_algorithm()

                    #Reconstruct path
                    current = goal
                    path = []
                    while current is not None and current in came_from:
                        path.append(current)
                        current = came_from[current]
                    path.reverse()

                    #Draw Djikstra's path
                    for col, row in path:
                        x = col * terrain.tile_size
                        y = row * terrain.tile_size
                        pygame.draw.rect(surface, YELLOW, (x, y, terrain.tile_size, terrain.tile_size), 0)

            #Space = Generate new map
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    terrain = TerrainMap()
                    graph = GridGraph(terrain)
                    terrain.render(surface)
                    clicks.clear()
                    path = []

            if event.type == pygame.QUIT:
                pygame.quit()
                exit()

        #Insert surface
        screen.blit(surface, (0, 0))

        #Draw clicks
        clicks.draw_clicks(screen)

        pygame.display.flip()
        clock.tick(60)

if __name__ == "__main__":
    main()
# ============================================================