# APES-plot2
import numpy as np
import matplotlib
matplotlib.use("Agg")
import matplotlib.pyplot as plt

# Formula: E7 = E6*EXP(r*(1-E6)), p_0 starts at 0.001 by default
def run(r, p0 = 0.001, n = 101):
    # let's just use a plain list first, append stuff, then convert to array later
    # maybe slightly slower than pre-allocating, but whatever, n=101 is tiny anyway
    p_list = [p0]
    
    for i in range(n - 1):
        prev = p_list[-1]
        next_val = prev * np.exp(r * (1 - prev))
        p_list.append(next_val)
        
    return np.array(p_list)

R1 = [0.2, 0.4, 0.7, 0.9]       # Part 3 - steady growth stuff
R2 = [1.75, 2.0, 3.0, 5.0]      # Part 4 - wild oscillations / chaos stuff
t = np.arange(101)

# Colors for the plots
BLUE_COLOR = "#5B9BD5"
YELLOW_COLOR = "#FFFF00"

def panel(ax, r, log_scale):
    # helper to plot individual subplots
    pop_data = run(r)
    
    ax.plot(t, pop_data, color=BLUE_COLOR, lw=1.8)
    ax.axhline(1, color=ORANGE_COLOR, ls="--", lw=2.5)   # K = 1 is the carrying capacity line
    
    ax.set_title(f"growth rate r = {r}", fontsize=12)
    ax.set_xlabel("time cycles")
    ax.set_ylabel("population / carrying capacity")
    ax.set_xlim(0, 100)
    ax.grid(alpha=0.35)
    
    if log_scale:
        ax.set_yscale("log")
    else:
        ax.set_ylim(0, 1.2)

# --- PART 3 PLOT ---
# linear y-axis for lower r values
fig1, axs1 = plt.subplots(2, 2, figsize=(10, 7.5))

# TODO: check if ravel works correctly here. Yes it should.
for ax, r_val in zip(axs1.ravel(), R1):
    panel(ax, r_val, False)

fig1.suptitle("Part 3: r = 0.2, 0.4, 0.7, 0.9 (dashed orange line = carrying capacity, K = 1)", fontsize=11)
fig1.tight_layout()
fig1.savefig("fig_part3.png", dpi=160)
# plt.close(fig1) # decided not to close, just leave it

# --- PART 4 PLOT ---
# log scale for higher r values because things blow up or oscillate wildly
fig2, axs2 = plt.subplots(2, 2, figsize=(10, 7.5))

for ax, r_val in zip(axs2.ravel(), R2):
    panel(ax, r_val, True)

fig2.suptitle("Part 4: r = 1.75, 2.0, 3.0, 5.0 (log10 y-axis; dashed orange line = K = 1)", fontsize=11)
fig2.tight_layout()
fig2.savefig("fig_part4.png", dpi=160)

print("Plots generated successfully. Now printing summary stats for the report table...")
print("-" * 65)

# Print out metrics for the homework table
all_r = R1 + R2
for r in all_r:
    p = run(r)
    
    # find when it hits 50% and 99% of K
    try:
        t50 = next(i for i, x in enumerate(p) if x >= 0.5)
    except StopIteration:
        t50 = "N/A"
        
    t99 = next((i for i, x in enumerate(p) if x >= 0.99), None)
    
    # check when it settles within 1% of 1.0
    settle = next((i for i in range(101) if np.all(abs(p[i:] - 1) < 0.01)), None)
    
    # how many times did it overshoot the carrying capacity?
    n_over = int(np.sum(p > 1.0001))
    
    print(f"r={r:<5} | t50: {str(t50):<4} | t99: {str(t99):<4} | settle1%: {str(settle):<4} | over: {n_over:<3} | max: {p.max():.3f}")

print("-" * 65)apes plot
