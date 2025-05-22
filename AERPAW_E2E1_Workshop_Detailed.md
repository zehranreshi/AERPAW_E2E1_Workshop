## I. Configuring the eNB + EPC on the UAV Node (Time: ~6 minutes)

Navigate to the radio scripts directory

```bash
cd /root/Profiles/ProfileScripts/Radio
```

Now we need to copy the default eNB+EPC script into the file that will be executed by the controller script:

```bash
cp Samples/startSRSRAN-SISO-EPCandENB.sh startRadio.sh
```

(This duplicates the sample EPC+eNB startup script as `startRadio.sh`, which is the standard script called by the master experiment launcher.)

To ensure it runs during the experiment:

```bash
nano /root/startexperiment.sh
```

(This opens the master script. You must uncomment the call to `startRadio.sh`.)

Scroll down and uncomment this line:

```bash
./Radio/startRadio.sh
```

Now set the UAV's flight control script:

```bash
cd /root/Profiles/ProfileScripts/Vehicle
cp Samples/startPreplannedTrajectory.sh startVehicle.sh
nano startVehicle.sh
```

Edit the following line in `startVehicle.sh` to reference the plan file of you choosing. You can make your own flight path by following part IV:

```bash
export MISSION=$PROFILE_DIR"/vehicle_control/PreplannedTrajectory/Missions/custom.plan"
```

## II. Configuring the Ground Nodes as UEs (Time: ~4 minutes)

SSH into the E-VM of each UE node and navigate to the script folder:

```bash
cd /root/Profiles/ProfileScripts/Radio
```

Copy the sample UE script to the standard file:

```bash
cp Samples/startSRSRAN-SISO-UE.sh startRadio.sh
```

Enable it in the experiment launch sequence:

```bash
nano /root/startexperiment.sh
```

Uncomment the line:

```bash
./Radio/startRadio.sh
```

Each UE script uses a node-specific IMSI value to ensure proper registration with the EPC. These are mapped to IP addresses in the EPC’s user_db.csv file.

## III. Starting the Experiment (Time: ~2 minutes)

Once every script is in place and `startexperiment.sh` is configured, use the following commands to start and monitor the experiment.

Start the experiment:

```bash
/root/startexperiment.sh
```

To monitor the experiment run tail -f on some of the results files:

```bash
tail -f /root/Results/<timestamp>_radio_enb_log.txt
```

This will print logs as they are added, allowing you to see if the eNB has started correctly.

# III. Running Traffic Tools (Time: ~5 minutes)

On the eNB (server side):

```bash
cd /root/Profiles/ProfileScripts/Traffic
cp Samples/startIperf.sh startTraffic.sh
nano /root/startexperiment.sh
```

Add:

```bash
./startTraffic.sh server ogstun
```

The first argument of `server` tells the script to run an Iperf server process. The second argument gives the name of the interface the server of client should connect to.

On each UE (client side):

```bash
cp Samples/startIperf.sh startTraffic.sh
nano /root/startexperiment.sh
```

Add:

```bash
./startTraffic.sh client tun_srsue
```

### MGEN

"On eNB (receiver):

```bash
cp Samples/startMgenRx.sh startTraffic.sh
```

On UE (sender):

```bash
cp Samples/startMgenTx.sh startTraffic.sh
```

(This creates continuous test traffic for performance analysis.)"

# VII. Viewing Results and Generating Graphs (Time: ~6 minutes)

"All logs are saved under `/root/Results/` on each node. These include channel stats (CQI, MCS, SNR, bitrate, etc.) in `srsENB.log` and `srsEPC.log`.

To graph throughput vs. UAV distance, use:

```python
import matplotlib.pyplot as plt
import pandas as pd

df = pd.read_csv("iperf_and_distance_log.csv")

fig, ax1 = plt.subplots()
ax1.scatter(df["time"], df["throughput_mbps"], label="Throughput", s=10)
ax1.set_ylabel("Throughput (Mbps)")
ax1.set_xlabel("Time (s)")

ax2 = ax1.twinx()
ax2.plot(df["time"], df["distance_from_LW1_m"], color="red")
ax2.set_ylabel("Distance from LW1 (m)")

plt.title("Throughput vs Distance")
plt.show()
```

This helps correlate mobility with channel performance."

# II. Mission Plan Creation Using QGroundControl (Time: ~6 minutes)

"Before we configure any nodes, we need to ensure that our UAV has a defined flight path. This is accomplished using a mission plan file, which instructs the UAV where to go, at what altitude, and at what speed. The recommended tool for this is QGroundControl. It's an open-source ground control station that provides a visual interface for creating and editing waypoint missions.

Here is how to create a mission file from scratch:

1. Open QGroundControl and click on the 'Plan' tab to switch to plan mode.
2. Use the map interface to right-click and add a 'Takeoff' waypoint. Set the altitude (e.g., 30m).
3. Continue clicking to add more waypoints along the desired path.
4. For each waypoint, you can set:
   - Altitude (within 30–90m for Phase 1)
   - Navigation speed (≤12 m/s)
   - Optional heading (0–359 degrees, where 0 = North)
5. To ensure compliance with allowed flight zones:
   - Load the Phase 1 Geofence by downloading the `.kml` file from AERPAW
   - Click 'Fence' → 'Polygon Fence' → 'Load kml/shp' to import it
   - Adjust waypoints to stay within bounds
6. Click File → Save As and name your mission file (e.g., `custom.plan`)
7. Transfer it to the UAV node via `scp` or MobaXterm.

Once uploaded to the E-VM, edit the startVehicle.sh file to point to the custom plan:

```bash
cd /root/Profiles/ProfileScripts/Vehicle
nano startVehicle.sh
```

In the file, change the mission path line to:

````bash
export MISSION=$PROFILE_DIR"/vehicle_control/PreplannedTrajectory/Missions/custom.plan"
```"
````
