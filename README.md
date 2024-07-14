```
import requests
import os
import time
from tqdm import tqdm

def fetch_versions():
    url = "https://gist.githubusercontent.com/osipxd/6119732e30059241c2192c4a8d2218d9/raw/3368c2b735b0b39310a193ed4109ebcf7914ab9f/paper-versions.json"
    response = requests.get(url)
    if response.status_code == 200:
        return response.json()
    else:
        print("Failed to fetch versions")
        return None

def prompt_user_for_version(versions):
    print("Available versions:")
    for version in versions.keys():
        print(version)
    
    selected_version = input("Enter the version you want to download: ")
    if selected_version in versions:
        return selected_version
    else:
        print("Invalid version selected")
        return None

def download_version(download_url, file_path):
    response = requests.get(download_url, stream=True)
    total_size = int(response.headers.get('content-length', 0))
    block_size = 1024  # 1 Kibibyte
    t = tqdm(total=total_size, unit='iB', unit_scale=True)
    with open(file_path, "wb") as file:
        for chunk in response.iter_content(chunk_size=block_size):
            t.update(len(chunk))
            file.write(chunk)
    t.close()
    if total_size != 0 and t.n != total_size:
        print("ERROR, something went wrong")
    else:
        print(f"Downloaded {file_path}")

def create_files(version):
    eula_content = """#By changing the setting below to TRUE you are indicating your agreement to our EULA (https://aka.ms/MinecraftEULA).
#Fri Jul 05 16:20:49 ICT 2024
eula=true
"""
    server_properties_content = """#Minecraft server properties
#Fri Jul 05 16:21:12 ICT 2024
accepts-transfers=false
allow-flight=false
allow-nether=true
broadcast-console-to-ops=true
broadcast-rcon-to-ops=true
bug-report-link=
debug=false
difficulty=easy
enable-command-block=false
enable-jmx-monitoring=false
enable-query=false
enable-rcon=false
enable-status=true
enforce-secure-profile=true
enforce-whitelist=false
entity-broadcast-range-percentage=100
force-gamemode=false
function-permission-level=2
gamemode=survival
generate-structures=true
generator-settings={}
hardcore=false
hide-online-players=false
initial-disabled-packs=
initial-enabled-packs=vanilla
level-name=world
level-seed=
level-type=minecraft\\:normal
log-ips=true
max-chained-neighbor-updates=1000000
max-players=20
max-tick-time=60000
max-world-size=29999984
motd=A Minecraft Server
network-compression-threshold=256
online-mode=false
op-permission-level=4
player-idle-timeout=0
prevent-proxy-connections=false
pvp=true
query.port=25565
rate-limit=0
rcon.password=
rcon.port=25575
region-file-compression=deflate
require-resource-pack=false
resource-pack=
resource-pack-id=
resource-pack-prompt=
resource-pack-sha1=
server-ip=
server-port=25565
simulation-distance=10
spawn-animals=true
spawn-monsters=true
spawn-npcs=true
spawn-protection=16
sync-chunk-writes=true
text-filtering-config=
use-native-transport=true
view-distance=10
white-list=false
"""
    run_bat_content = f"""CHCP 65001
@echo off
java -Xms1G -Xmx1G -XX:+UseG1GC -jar paper-{version}.jar -nogui
pause
"""

    with open("eula.txt", "w") as file:
        file.write(eula_content)
    with open("server.properties", "w") as file:
        file.write(server_properties_content)
    with open("run.bat", "w") as file:
        file.write(run_bat_content)
    print("Created eula.txt, server.properties, and run.bat files")

def main():
    versions_data = fetch_versions()
    if not versions_data:
        return

    selected_version = prompt_user_for_version(versions_data["versions"])
    if not selected_version:
        return

    download_url = versions_data["versions"][selected_version]
    file_path = os.path.join(os.getcwd(), f"paper-{selected_version}.jar")

    download_version(download_url, file_path)
    create_files(selected_version)

if __name__ == "__main__":
    main()

```
