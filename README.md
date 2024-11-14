## Load Capacity Analysis Code

import math

# Constants and parameters
diameter = 0.00159  # Diameter of wire in meters
force = 108  # Applied force in Newtons
K = 3  # Stress concentration factor
width = 0.001  # Width of finger segments in meters (1 mm)
height = 0.001  # Height of finger segments in meters (1 mm)
load_fingertip = 1.0  # Load applied at fingertip in Newtons
safety_factor = 1.5  # Safety factor for torque calculation

# MG996R servo torque capacity
servo_torque_capacity = 1.08  # N-m

# Finger segment lengths in meters
finger_segments = {
    "Index": [4.5 * 0.001, 1.2 * 0.001, 3.0 * 0.001],
    "Middle": [5.5 * 0.001, 1.7 * 0.001, 1.7 * 0.001],
    "Ring": [2.0 * 0.001, 1.2 * 0.001, 3.0 * 0.001],
    "Pinky": [3.0 * 0.001, 3.0 * 0.001, 3.0 * 0.001],
    "Thumb": [4.0 * 0.001, 3.5 * 0.001]
}

# Wire Stress Analysis

# Calculating cross-sectional area
area = math.pi * (diameter / 2) ** 2
# Calculating nominal stress
nominal_stress = force / area
# Adjusted stress with concentration factor
adjusted_stress = K * nominal_stress

# Bending Stress in Finger Segments

# Moment of inertia for a rectangular cross-section
I = (1/12) * width * (height ** 3)
# Calculate bending stress for each segment under a 1N load at the fingertip
bending_stresses = {}

for finger, segments in finger_segments.items():
    bending_moment = 0
    stresses = []
    for length in reversed(segments):
        bending_moment += load_fingertip * length
        bending_stress = (bending_moment * (height / 2)) / I
        stresses.insert(0, bending_stress)
    bending_stresses[finger] = stresses

# Servo Torque Verification

# Function to calculate cumulative torque per finger
def calculate_finger_torque(segments, load):
    total_torque = 0
    cumulative_load = load
    for length in reversed(segments):
        segment_torque = cumulative_load * length
        total_torque += segment_torque
        cumulative_load += load  # Increase load cumulatively
    return total_torque

# Calculating torque required for each finger
finger_torques = {finger: calculate_finger_torque(segments, load_fingertip) for finger, segments in finger_segments.items()}
# Apply safety factor to each torque
adjusted_finger_torques = {finger: torque * safety_factor for finger, torque in finger_torques.items()}

# Calculating the maximum practical load capacity based on the most demanding adjusted torque
max_adjusted_torque = max(adjusted_finger_torques.values())
practical_load_capacity = (servo_torque_capacity / max_adjusted_torque) * load_fingertip

# Display of results
print("=== Key Findings ===")

# 1. Wire Stress Analysis Summary
print("\nWire Stress Analysis:")
print(f"Cross-Sectional Area (m^2): {area:.6e}")
print(f"Nominal Stress (Pa): {nominal_stress:.2f}")
print(f"Adjusted Stress with Stress Concentration (Pa): {adjusted_stress:.2f}")

# 2. Bending Stress Summary for each finger segment
print("\nBending Stress per Finger Segment (Pa):")
for finger, stresses in bending_stresses.items():
    print(f"{finger} Finger: {['{:.2e}'.format(s) for s in stresses]}")

# 3. Servo Torque Verification
print("\nServo Torque per Finger (N-m):")
for finger, torque in finger_torques.items():
    print(f"{finger} Finger: {torque:.4f} N-m")

print("\nAdjusted Servo Torque per Finger with Safety Factor (N-m):")
for finger, adj_torque in adjusted_finger_torques.items():
    print(f"{finger} Finger: {adj_torque:.4f} N-m")

# Practical Load Capacity based on servo limitation
print("\nPractical Load Capacity:")
print(f"Maximum Practical Load Capacity (N): {practical_load_capacity:.2f} N")
print(f"Maximum Practical Load Capacity (lb): {practical_load_capacity / 4.448:.2f} lb")  # Converting N to pounds

