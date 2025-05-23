## I. Configuring the gNB

Navigate to the radio scripts directory

```bash
cd /root/Profiles/ProfileScripts/Radio
```

Now copy the script to run the OAI gNB and 5G core network:

```bash
cp Samples/startOpen5GS-OAI-gNB.sh ./startRadio.sh
```

To ensure it runs during the experiment:

```bash
nano /root/startexperiment.sh
```

(This opens the master script. You must uncomment the call to `startRadio.sh`.)

Scroll down and uncomment this line:

```bash
./Radio/startRadio.sh
```

## II. Configuring the IQ collector Node

SSH into the E-VM of each UE node and navigate to the script folder:

```bash
cd /root/Profiles/ProfileScripts/Radio
```

Copy the sample UE script to the standard file:

```bash
cp Samples/start_5G_IQ_Collection.sh startRadio.sh
```

Enable it in the experiment launch sequence:

```bash
nano /root/startexperiment.sh
```

Uncomment the line:

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

## III. Starting the Experiment (Time: ~2 minutes)

Once every script is in place and `startexperiment.sh` is configured, use the following commands to start and monitor the experiment.

Start the experiment:

```bash
/root/startexperiment.sh
```

To monitor the experiment (on the gNB side) run tail -f on some of the results files:

```bash
tail -f /root/Results/<timestamp>_radio_enb_log.txt
```

This will print logs as they are added, allowing you to see if the eNB has started correctly.

# IV. Viewing Results and Generating Graphs (Time: ~6 minutes)

run the log2csv script on the output of the vehicle:

```bash
python3 ./Profile/PostProcessing/log2csv.py /root/Results/<timestamp>_vehicle_out.txt -m vehicleLog -o vehicle.csv
```

next we run the CSV merge script to combine the two resulting csv files based on their timestamps:

```bash
python3 ./Profile/PostProcessing/csvMerge/csvMerge.py vehicle.csv /root/Results/<timestamp>_iq_out.csv --output merged_result.csv
```

Finally this script will create a kml file that can be viewed in google earth:

```bash
python3 ./Profile/PostProcessing/akmlGen.py merged_result.csv --colormap jet_r --output result.kml
```
