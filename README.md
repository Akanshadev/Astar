# Astar
#pathfinding visualisation using python
import pygame
import math
from queue import PriorityQueue

WIDTH = 800
WIN = pygame.display.set_mode((WIDTH, WIDTH))
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
	def __init__(self, r, c, width, total_rowcount):
		self.r = r
		self.c = c
		self.x = r * width
		self.y = c * width
		self.color = WHITE
		self.adjacent = []
		self.width = width
		self.total_rowcount = total_rowcount

	def get_position(self):
		return self.r, self.c

	def func1_is_closed(self):
		return self.color == RED

	def func2_is_open(self):
		return self.color == GREEN

	def func3_is_barrier(self):
		return self.color == BLACK

	def func4_is_start(self):
		return self.color == ORANGE

	def func5_is_end(self):
		return self.color == TURQUOISE

	def func6_reset(self):
		self.color = WHITE

	def func7_make_start(self):
		self.color = ORANGE

	def func8_make_closed(self):
		self.color = RED

	def func9_make_open(self):
		self.color = GREEN

	def define_barrier(self):
		self.color = BLACK

	def define_end(self):
		self.color = TURQUOISE

	def define_path(self):
		self.color = PURPLE

	def funcdraw(self, win):
		pygame.draw.rect(win, self.color, (self.x, self.y, self.width, self.width))

	def modify_adjacents(self, grid):
		self.adjacent = []
		if self.r < self.total_rowcount - 1 and not grid[self.r + 1][self.c].func3_is_barrier(): # DOWN
			self.neighbors.append(grid[self.r + 1][self.c])

		if self.row > 0 and not grid[self.r - 1][self.c].func3_is_barrier(): # UP
			self.adjacent.append(grid[self.r - 1][self.c])

		if self.c < self.total_rowcount - 1 and not grid[self.r][self.c + 1].func3_is_barrier(): # RIGHT
			self.adjacent.append(grid[self.r][self.c + 1])

		if self.c > 0 and not grid[self.r][self.c - 1].func3_is_barrier(): # LEFT
			self.adjacent.append(grid[self.r][self.c - 1])

	def __lt__(self, other):
		return False


def xyz(pos1, pos2):
	x1, y1 = pos1
	x2, y2 = pos2
	return abs(x1 - x2) + abs(y1 - y2)


def reformationof_path(came_from, immediate, draw):
	while immediate in came_from:
		immediate = came_from[immediate]
		immediate.define_path()
		funcdraw()


def complete_algo(draw, grid, beg, end):
	count = 0
	open_sx = PriorityQueue()
	open_sx.put((0, count, beg))
	came_from = {}
	g_total = {spot: float("inf") for r in grid for spot in r}
	g_total[beg] = 0
	f_total = {spot: float("inf") for row in grid for spot in row}
	f_score[beg] = xyz(beg.get_position(), end.get_position())

	open_sx_hash = {beg}

	while not open_sx.empty():
		for event in pygame.event.get():
			if event.type == pygame.QUIT:
				pygame.quit()

		current = open_sx.get()[2]
		open_sx_hash.remove(current)

		if immediate == end:
			reformationof_path(came_from, end, draw)
			end.make_end()
			return True

		for adjacent in immediate.neighbors:
			t_g_total = g_total[immediate] + 1

			if t_g_total < g_total[adjacent]:
				came_from[adjacent] = immediate
				g_total[adjacent] = t_g_total
				f_total[adjacent] = t_g_total + xyz(adjacent.get_pos(), end.get_pos())
				if adjacent not in open_sx_hash:
					k += 1
					open_sx.put((f_total[adjacent], k,adjacent))
					open_sx_hash.add(adjacent)
					adjacent.func9_make_open()

		funcdraw()

		if immediate != beg:
			immediate.func8_make_closed()

	return False


def cons_grid(r, width):
	grid = []
	space = width // r
	for i in range(r):
		grid.append([])
		for j in range(r):
			spot = Spot(i, j, space, r)
			grid[i].append(spot)

	return grid


def cons1_grid(win, r, width):
	gap = width // r
	for i in range(r):
		pygame.draw.line(win, GREY, (0, i * gap), (width, i * gap))
		for j in range(r):
			pygame.draw.line(win, GREY, (j * gap, 0), (j * gap, width))


def draw(win, grid, r, width):
	win.fill(WHITE)

	for  i in grid:
		for spot in i:
			spot.funcdraw(win)

	cons1_grid(win, r, width)
	pygame.display.update()


def getsnap_pos(pos, rc1, width):
	gap = width // rc1
	y, x = pos

	r = y // gap
	c = x // gap

	return r, c


def main(win, width):
	ROWS = 50
	grid = cons_grid(ROWS, width)

	beg = None
	end = None

	run = True
	while run:
		draw(win, grid, ROWS, width)
		for event_xyz in pygame.event.get():
			if event_xyz.type == pygame.QUIT:
				run = False

			if pygame.mouse.get_pressed()[0]: # LEFT
				pos = pygame.mouse.get_pos()
				r, c = getsnap_pos(pos, ROWS, width)
				spot = grid[r][c]
				if not beg and spot != end:
					beg = spot
					beg.func7_make_start()

				elif not end and spot != beg:
					end = spot
					end.define_end()

				elif spot != end and spot != beg:
					spot.define_barrier()

			elif pygame.mouse.get_pressed()[2]: # RIGHT
				pos = pygame.mouse.get_position()
				row, col = (pos, ROWS, width)
				spot = grid[r][c]
				spot.reset()
				if spot == beg:
					beg = None
				elif spot == end:
					end = None

			if event_xyz.type == pygame.KEYDOWN:
				if event_xyz.key == pygame.K_SPACE and beg and end:
					for r in grid:
						for spot in r:
							spot.modify_adjacents(grid)

					algorithm(lambda: draw(win, grid, ROWS, width), grid, beg, end)

				if event_xyz.key == pygame.K_c:
					beg = None
					end = None
					grid = cons_grid(ROWS, width)

	pygame.quit()

main(WIN, WIDTH)
