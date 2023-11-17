import tkinter as tk
from tkinter import messagebox
from bs4 import BeautifulSoup
import requests

def is_valid(grid, r, c, k):
    not_in_row = k not in grid[r]
    not_in_column = k not in [grid[i][c] for i in range(9)]
    not_in_box = k not in [grid[i][j] for i in range(r//3*3, r//3*3+3) for j in range(c//3*3, c//3*3+3)]
    return not_in_row and not_in_column and not_in_box

def solve(grid, r=0, c=0):
    if r == 9:
        return True
    elif c == 9:
        return solve(grid, r+1, 0)
    else:
        next_r, next_c = r, c+1
        if next_c == 9:
            next_r, next_c = r+1, 0

        if grid[r][c] != 0:
            return solve(grid, next_r, next_c)
        else:
            for k in range(1, 10):
                if is_valid(grid, r, c, k):
                    grid[r][c] = k
                    if solve(grid, next_r, next_c):
                        return True
                    grid[r][c] = 0
            return False

def fetch_sudoku_grid(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    
    # Extract the Sudoku grid from the website
    grid = []
    for row in soup.find_all('tr')[:9]:  # Limit to the first 9 rows
        grid.append([int(cell.text) if cell.text.isdigit() else 0 for cell in row.find_all('td')[:9]])  # Limit to the first 9 cells
    
    return grid

class SudokuSolverApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Sudoku Solver App")

        self.grid_entries = [[None]*9 for _ in range(9)]

        # Create entry widgets for manual Sudoku input
        for i in range(9):
            for j in range(9):
                entry = tk.Entry(root, width=2, font=('Arial', 16), justify='center')
                entry.grid(row=i, column=j)
                self.grid_entries[i][j] = entry

        # Create "Solve" button
        solve_button = tk.Button(root, text="Solve", command=self.solve_sudoku)
        solve_button.grid(row=10, column=4)

    def solve_sudoku(self):
        # Get the input Sudoku grid
        input_grid = [[int(entry.get()) if entry.get().isdigit() else 0 for entry in row] for row in self.grid_entries]

        # Solve the Sudoku puzzle
        if solve(input_grid):
            # Display the solution in the GUI
            for i in range(9):
                for j in range(9):
                    self.grid_entries[i][j].delete(0, tk.END)
                    self.grid_entries[i][j].insert(0, input_grid[i][j])
        else:
            messagebox.showinfo("Sudoku Solver", "No solution found for the given Sudoku puzzle.")

if __name__ == "__main__":
    root = tk.Tk()
    app = SudokuSolverApp(root)
    root.mainloop()
