import streamlit as st
import numpy as np
import matplotlib.pyplot as plt
import time

# =====================================
# PAGE CONFIG
# =====================================
st.set_page_config(page_title="Cricket Ball Physics Simulator", layout="wide")
st.title("🏏 Cricket Ball Physics Simulator")
st.markdown("Interactive simulation of swing, spin, bounce, pitch zones, and stumps.")

g = 9.8
dt = 0.02

# =====================================
# BALL PRESETS (m/s)
# =====================================
ball_presets = {
    "Inswing":        (25, 5, -0.6, 0.0, 0.65),
    "Outswing":       (25, 5, 0.6, 0.0, 0.65),
    "Reverse Swing":  (27, 5, -1.0, 0.0, 0.65),
    "Slow Ball":      (18, 7, 0.0, 0.0, 0.60),
    "Leg Cutter":     (24, 6, 0.0, -1.2, 0.65),
    "Off Cutter":     (24, 6, 0.0, 1.2, 0.65),
    "Yorker":         (30, 2, 0.0, 0.0, 0.55),
    "Bouncer":        (28, 15, 0.0, 0.0, 0.90),
    "Off Spin":       (20, 6, 0.0, 1.8, 0.70),
    "Leg Spin":       (20, 6, 0.0, -1.8, 0.70),
    "Top Spin":       (22, 8, 0.0, 0.0, 1.05),
    "Flipper":        (23, 4, 0.0, 0.5, 0.40),
    "Carrom Ball":    (21, 6, 0.0, 2.5, 0.75),
}

pitch_types = {
    "Normal": (1.0, 1.0),
    "Green":  (1.1, 0.8),
    "Dusty":  (0.9, 1.5),
    "Wet":    (0.7, 0.6),
}

# =====================================
# USER INPUT
# =====================================
col1, col2 = st.columns(2)
with col1:
    selected_ball = st.selectbox("Select Ball Type", list(ball_presets.keys()))
with col2:
    selected_pitch = st.selectbox("Select Pitch Type", list(pitch_types.keys()))

# Get default ball parameters
speed_mps, angle_deg, swing, spin, bounce = ball_presets[selected_ball]
bounce_mult, spin_mult = pitch_types[selected_pitch]

bounce *= bounce_mult
spin *= spin_mult

# -----------------------------
# Sidebar sliders
# -----------------------------
st.sidebar.header("🎛 Adjust Parameters")

# Convert speed to km/h for slider
speed_kmh_default = int(speed_mps * 3.6)
speed_kmh = st.sidebar.slider(
    "Speed (km/h)", 30, 150, speed_kmh_default, step=1,
    help="Typical cricket ball speeds"
)

# Convert back to m/s for calculations
speed = speed_kmh / 3.6

angle_deg = st.sidebar.slider("Release Angle (degrees)", 0, 20, int(angle_deg))
swing = st.sidebar.slider("Swing (air movement)", -2.0, 2.0, float(swing), step=0.1)
spin = st.sidebar.slider("Spin (after bounce)", -3.0, 3.0, float(spin), step=0.1)
bounce = st.sidebar.slider("Bounce Coefficient", 0.0, 1.5, float(bounce), step=0.05)

# Sidebar animation speed
st.sidebar.header("⚡ Animation Control")
animation_speed = st.sidebar.slider(
    "Animation Speed", 1, 50, 10, step=1,
    help="Higher = slower animation, lower = faster animation"
)

angle = np.radians(angle_deg)

# =====================================
# TRAJECTORY CALCULATION
# =====================================
x = 0
y = 1.8
z = 0

vx = speed * np.cos(angle)
vy = speed * np.sin(angle)
vz = 0

x_vals, y_vals, z_vals = [], [], []
bounced = False
bounce_x_location = None

while x < 40 and y >= 0:
    if not bounced:
        vz += swing * dt
    vy -= g * dt
    x += vx * dt
    y += vy * dt
    z += vz * dt

    if y <= 0 and not bounced:
        y = 0
        vy = -vy * bounce
        vz += spin
        bounced = True
        bounce_x_location = x

    x_vals.append(x)
    y_vals.append(y)
    z_vals.append(z)

# =====================================
# PLOT FUNCTION (text removed)
# =====================================
def plot_field(ax, top_view=False, pitch_height=1.5):
    ax.set_facecolor("#2E8B57" if top_view else "#3CB043")
    ax.add_patch(plt.Rectangle((0, -1 if top_view else 0), 40, 2 if top_view else pitch_height, color="#C2B280", zorder=0))

    if not top_view:
        # Pitch zones (no text labels)
        ax.add_patch(plt.Rectangle((10,0), 5, pitch_height, color='orange', alpha=0.2))
        ax.add_patch(plt.Rectangle((15,0), 3, pitch_height, color='yellow', alpha=0.2))
        ax.add_patch(plt.Rectangle((18,0), 2, pitch_height, color='red', alpha=0.2))

    # Batter / crease
    ax.axvline(20, linestyle="--", linewidth=2, color="white")
    if not top_view:
        ax.axhline(0, linewidth=3, color="white")
        ax.text(20, 18, "Batter", rotation=90, va='top', color='white', fontsize=10)

    # Stumps
    stump_height = 0.71
    stump_width = 0.03
    stump_x = 20
    stump_positions = [-stump_width, 0, stump_width]
    for pos in stump_positions:
        if not top_view:
            ax.plot([stump_x + pos]*2, [0, stump_height], color='brown', linewidth=2)
        else:
            ax.scatter(stump_x + pos, 0, color='brown', s=50)

    # Top view crease lines
    if top_view:
        ax.axhline(-1, linestyle="--", color="white", alpha=0.7)
        ax.axhline(1, linestyle="--", color="white", alpha=0.7)

    ax.set_xlim(0,40)
    ax.set_ylim(-5 if top_view else 0, 5 if top_view else 20)
    ax.tick_params(colors='white', labelsize=8)
    ax.grid(True, linestyle='--', alpha=0.3)
    return ax

# =====================================
# ANIMATION SIDE-BY-SIDE
# =====================================
st.subheader("📈 Ball Trajectory (Side & Top View)")
frame_step = max(1, int(animation_speed / 2))

col1, col2 = st.columns(2)
placeholder1 = col1.empty()
placeholder2 = col2.empty()

for i in range(0, len(x_vals), frame_step):
    # Side view
    with col1:
        fig1, ax1 = plt.subplots(figsize=(3,2))
        ax1 = plot_field(ax1, top_view=False)
        ax1.plot(x_vals[:i+1], y_vals[:i+1], color='red', linewidth=2)
        ax1.scatter(x_vals[i], y_vals[i], s=50, color='red', edgecolors='black')
        if bounce_x_location:
            ax1.scatter(bounce_x_location, 0, s=40, color='yellow', edgecolors='black')
            ax1.text(bounce_x_location, 1.2, "Bounce", ha="center", color='white', fontsize=8)
        ax1.set_title("Side View", color='white', fontsize=10)
        ax1.set_xlabel("Distance (m)", color='white', fontsize=8)
        ax1.set_ylabel("Height (m)", color='white', fontsize=8)
        placeholder1.pyplot(fig1)
        plt.close(fig1)

    # Top view
    with col2:
        fig2, ax2 = plt.subplots(figsize=(3,2))
        ax2 = plot_field(ax2, top_view=True)
        ax2.plot(x_vals[:i+1], z_vals[:i+1], color='red', linewidth=2)
        ax2.scatter(x_vals[i], z_vals[i], s=50, color='red', edgecolors='black')
        if bounce_x_location:
            bounce_index = min(range(len(x_vals)), key=lambda j: abs(x_vals[j]-bounce_x_location))
            ax2.scatter(bounce_x_location, z_vals[bounce_index], s=40, color='yellow', edgecolors='black')
            ax2.text(bounce_x_location, z_vals[bounce_index], "Bounce", color='white', fontsize=8)
        ax2.set_title("Top View", color='white', fontsize=10)
        ax2.set_xlabel("Distance (m)", color='white', fontsize=8)
        ax2.set_ylabel("Lateral (m)", color='white', fontsize=8)
        placeholder2.pyplot(fig2)
        plt.close(fig2)

    time.sleep(0.001)

# =====================================
# METRICS
# =====================================
st.subheader("📊 Ball Metrics")
col1, col2, col3 = st.columns(3)
with col1:
    st.metric("Total Distance", f"{round(x_vals[-1],2)} m")
with col2:
    if bounce_x_location:
        st.metric("Bounce Distance", f"{round(bounce_x_location,2)} m")
    else:
        st.metric("Bounce Distance", "No Bounce")
with col3:
    st.metric("Max Height", f"{round(max(y_vals),2)} m")

st.markdown(f"""
### 🧠 Physics Concepts
- Swing affects the ball in the air (top view before bounce).
- Spin affects the ball after bounce (top view after bounce).
- Bounce coefficient simulates pitch hardness.
- Speed and angle determine length and trajectory.
- Pitch zones (colored) match real distances without text labels.
- Crease lines and stumps illustrate batter end for presentation.
- Animation speed slider controls the speed of the simulation.
- Ball speed slider now uses **km/h**, typical in cricket broadcasts. Current speed: **{speed_kmh} km/h**
""")
