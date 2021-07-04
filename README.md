import pygame

import math
from queue import PriorityQueue

width = 800
WIN = pygame.display.set_mode((width, width))
pygame.display.set_caption("A* Path Finding Algorithm")

RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 255, 0)
YELLOW = (255, 255, 0)
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
PURPLE = (128, 0, 128)
ORANGE = (255, 165 ,0)
GREY = (128, 128, 128)
TURQUOISE = (64, 224, 208)

class Spot:
	def __init__(self, r, c, width, total_rs):
		self.r = r 
		self.c = c
		self.x = r * width
		self.y = c * width
		self.cor = WHITE
		self.neighbors = []
		self.width = width
		self.total_rs = total_rs

	def get_pos(self):
		return self.r, self.c

	def is_closed(self):
		return self.cor == RED

	def is_open(self):
		return self.cor == GREEN

	def is_barrier(self):
		return self.cor == BLACK

	def is_start(self):
		return self.cor == ORANGE

	def is_end(self):
		return self.cor == TURQUOISE

	def reset(self):
		self.cor = WHITE

	def make_start(self):
		self.cor = ORANGE

	def make_closed(self):
		self.cor = RED

	def make_open(self):
		self.cor = GREEN

	def make_barrier(self):
		self.cor = BLACK

	def make_end(self):
		self.cor = TURQUOISE

	def make_path(self):
		self.cor = PURPLE

	def draw(self, win):
		pygame.draw.rect(win, self.cor, (self.x, self.y, self.width, self.width))

	def update_neighbors(self, grid):
		self.neighbors = []
		if self.r < self.total_rs - 1 and not grid[self.r + 1][self.c].is_barrier(): # DOWN
			self.neighbors.append(grid[self.r + 1][self.c])

		if self.r > 0 and not grid[self.r - 1][self.c].is_barrier(): # UP
			self.neighbors.append(grid[self.r - 1][self.c])

		if self.c < self.total_rs - 1 and not grid[self.r][self.c + 1].is_barrier(): # RIGHT
			self.neighbors.append(grid[self.r][self.c + 1])

		if self.c > 0 and not grid[self.r][self.c - 1].is_barrier(): # LEFT
			self.neighbors.append(grid[self.r][self.c - 1])

	def __lt__(self, other):
		return False


def h(p1, p2):
	x1, y1 = p1
	x2, y2 = p2
	return abs(x1 - x2) + abs(y1 - y2)


def reconstruct_path(came_from, cur, draw):
	while cur in came_from:
		cur = came_from[cur]
		cur.make_path()
		draw()


def algorithm(draw, grid, start, end):
	count = 0
	open_set = PriorityQueue()
	open_set.put((0, count, start))
	came_from = {}
	g_score = {spot: float("inf") for r in grid for spot in r}
	g_score[start] = 0
	f_score = {spot: float("inf") for r in grid for spot in r}
	f_score[start] = h(start.get_pos(), end.get_pos())

	open_set_hash = {start}

	while not open_set.empty():
		for event in pygame.event.get():
			if event.type == pygame.QUIT:
				pygame.quit()

		cur = open_set.get()[2]
		open_set_hash.remove(cur)

		if cur == end:
			reconstruct_path(came_from, end, draw)
			end.make_end()
			return True

		for neighbor in cur.neighbors:
			temp_g_score = g_score[cur] + 1

			if temp_g_score < g_score[neighbor]:
				came_from[neighbor] = cur
				g_score[neighbor] = temp_g_score
				f_score[neighbor] = temp_g_score + h(neighbor.get_pos(), end.get_pos())
				if neighbor not in open_set_hash:
					count += 1
					open_set.put((f_score[neighbor], count, neighbor))
					open_set_hash.add(neighbor)
					neighbor.make_open()

		draw()

		if cur != start:
			cur.make_closed()

	return False


def make_grid(rs, width):
	grid = []
	gap = width // rs
	for i in range(rs):
		grid.append([])
		for j in range(rs):
			spot = Spot(i, j, gap, rs)
			grid[i].append(spot)

	return grid


def draw_grid(win, rs, width):
	gap = width // rs
	for i in range(rs):
		pygame.draw.line(win, GREY, (0, i * gap), (width, i * gap))
		for j in range(rs):
			pygame.draw.line(win, GREY, (j * gap, 0), (j * gap, width))


def draw(win, grid, rs, width):
	win.fill(WHITE)

	for r in grid:
		for spot in r:
			spot.draw(win)

	draw_grid(win, rs, width)
	pygame.display.update()


def get_clicked_pos(pos, rs, width):
	gap = width // rs
	y, x = pos

	r = y // gap
	c = x // gap

	return r, c


def main(win, width):
	rS = 50
	grid = make_grid(rS, width)

	start = None
	end = None

	run = True
	while run:
		draw(win, grid, rS, width)
		for event in pygame.event.get():
			if event.type == pygame.QUIT:
				run = False

			if pygame.mouse.get_pressed()[0]: # LEFT
				pos = pygame.mouse.get_pos()
				r, c = get_clicked_pos(pos, rS, width)
				spot = grid[r][c]
				if not start and spot != end:
					start = spot
					start.make_start()

				elif not end and spot != start:
					end = spot
					end.make_end()

				elif spot != end and spot != start:
					spot.make_barrier()

			elif pygame.mouse.get_pressed()[2]: # RIGHT
				pos = pygame.mouse.get_pos()
				r, c = get_clicked_pos(pos, rS, width)
				spot = grid[r][c]
				spot.reset()
				if spot == start:
					start = None
				elif spot == end:
					end = None

			if event.type == pygame.KEYDOWN:
				if event.key == pygame.K_SPACE and start and end:
					for r in grid:
						for spot in r:
							spot.update_neighbors(grid)

					algorithm(lambda: draw(win, grid, rS, width), grid, start, end)

				if event.key == pygame.K_c:
					start = None
					end = None
					grid = make_grid(rS, width)

	pygame.quit()

main(WIN, width)
