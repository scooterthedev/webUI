![Screenshot 2024-04-24 111145](https://raw.githubusercontent.com/scooterthedev/webUI/main/325336435-90b244ad-0040-4eed-984c-4cef83d6615e.png)
	

# X1Plus WebUI
 Dedicated and self served web based UI for Bambu printers running on the X1Plus firmware.

PreReqs:
1) X1Plus Firmware installed on your X1C Printer (https://github.com/X1Plus/X1Plus/wiki/Installation-Guide)

# How to use:
1) Run `webui-aiohttp-installer.py` on your client device. Input the printer login paramaters in the message box. Then wait for the necessary files to transfer over, and for the rest of the scripts to run.
1) Restart the printer
1) Enjoy your web UI on port 5001

# Quick Code References:
The mechanism used to send G-code commands to the printer involves several methods, primarily through MQTT and SSH commands. The process is initiated in different parts of the codebase, depending on the context and the specific action being performed.
1. MQTT for G-code Commands:
G-code commands are sent using MQTT in various macros. This is evident in the send_gcode function within macros like homexyz.py, x_travel_10.py, and others. The mqtt_pub function is called with a JSON payload containing the G-code command.
---
def send_gcode(gcode_line):

    json_payload = {
    
        "print": {
    
            "command": "gcode_line",
    
            "sequence_id": "2001",
    
            "param": gcode_line
    
        }
    
    }
    
    mqtt_pub(json.dumps(json_payload))



def mqtt_pub(message):

    command = f"source /usr/bin/mqtt_access.sh; mqtt_pub '{message}'"
    
    try:
    
        subprocess.run(command, shell=True, check=True, executable='/bin/bash')
    
    except subprocess.CalledProcessError as e:
---

2. SSH for Remote Command Execution:
SSH is used in the BambuHMI SSH Commander to execute Python scripts on the printer, which in turn send G-code commands. This is seen in the commands.py file where SSH commands are used to execute macros like x_travel_-10.py.

---
def MoveXNegative(self):

        cmd = "/opt/python/bin/python3 /mnt/sdcard/x1plus/macros/x_travel_-10.py"
    
        if self.ssh_client:
    
            command_thread = threading.Thread(target=self.execute_command, args=(cmd,))
    
            command_thread.start()
    
        else:
    
            self.append_output("Not connected to any server.")
---
3. Direct G-code Command Sending:
Direct sending of G-code commands is also facilitated through a DDS (Data Distribution Service) publisher in send_gcode.py, where commands are sent over a DDS network.
---
def send_command(pub, cmd):

    """Sends a command using DDS publisher."""
    
    msg = {"command": "gcode_line", "param": cmd, "sequence_id": 0}
    
    json_msg = json.dumps(msg)
    
    pub(json_msg)
    
    print(f"Command sent: {cmd}")



def main():

    pub = dds.publisher('device/request/print')
    
    time.sleep(3) 
    
    while True:
    
        cmd = input("Enter the gcode command you want to send or type 'exit' to quit: ")
    
        if cmd.lower() == 'exit':
    
            print("Exiting command input.")
    
            break
    
        send_command(pub, cmd)
---
4. Shell Scripts:
Shell scripts like heatbed_set.sh are used to send G-code commands via MQTT, which can be triggered through SSH or other mechanisms.
---
function set_heatbed_temp() {

    temp=$1
    
    json="{ \
    
        \"print\":{\"command\":\"gcode_line\", \"sequence_id\":\"2002\", \"param\":\"M140 S$temp\"} \
    
    }"
---
