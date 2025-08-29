# Orange-Data-Mining  
### (General notes)  

## Step-by-Step Instructions to Dockerize Orange3 with GUI

### 1. Create a Dockerfile for Orange3

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Consider Installing SQLite3 CLI to access .db, convert .db to .csv

# Install system dependencies for Qt and X11
RUN apt-get update && \ 
    apt-get install -y \
    libx11-6 \
    libxext6 \
    libxrender1 \
    libsm6 \
    libxrandr2 \
    libgl1 \
    libglu1-mesa \
    libglib2.0-0 \
    libqt5widgets5 \
    libqt5gui5 \
    libqt5core5a \
    xauth \
    x11-xserver-utils \
    && rm -rf /var/lib/apt/lists/*

# Install Orange3 and PyQt5 via pip
RUN pip install --no-cache-dir ex

CMD ["python", "-m", "Orange.canvas"]
```
### 2. Build the Docker image

```
docker build -f Dockerfile.orange -t my-orange3 .
```

### 3. Prepare your host environment

- Confirm your X server is running 
```
xhost
```
- Confirm terminal env $DISPLAY is set (usually :0 on Linux)
```
echo $DISPLAY
```
- Allow Docker containers to connect to X server:
```
xhost +local:docker
```		
- Ensure /tmp/.X11-unix has correct permissions (drwxrwxrwt):
```
ls -ld /tmp/.X11-unix
chmod 1777 /tmp/.X11-unix   # (may require sudo)
```
### 4. Run the Orange3 container with X11 forwarding

```
docker run --rm -it \                            
          -e DISPLAY=$DISPLAY \
          -v /tmp/.X11-unix:/tmp/.X11-unix \
          -v "$(pwd)/workspace:/app/workspace" \
          my-orange3
```
Note:  
    - X11 is uppercase  
	- Environment variable DISPLAY tells you where the window is rendered  
	- X11-unix socket directory from host is mounted to container  
	- And X11-unix needs correct permission, see 3.

### 5. If Orange.canvas gui didn't start  

- Inside the container, check to see if Orange3 is installed.

```
python -c "import Orange; print(Orange.__version__)"
```  

- Run Orange3 data mining tool GUI (Orange.canvas)

```
python -m Orange.canvas
```  

### 6. Cleanup after use  

- Revoke X server access for Docker containers when done:

```
xhost -local:docker
```

## Why xhost, xeyes, and permissions?

### X server:  
The tool/service that handles window rendering (e.g., Xorg, XQuartz)

### X11 socket (/tmp/.X11-unix):  
Unix domain socket directory used by the X server to communicate.  
Your container a) must mount this directory and have b) write permission to connect.  
- a) In docker run, -v /tmp/.X11-unix:/tmp/.X11-unix  
- b) chmod 1777 /tmp/.X11-unix  
	
### DISPLAY env:  
Tells GUI apps where to display the window (e.g., :0 = local screen)  
	
### xhost:  
Command-line tool to grant access to your X server for other users/apps  

- e.g. xhost +local:docker   
    temporarily allows Docker containers (local users) to connect and display GUIs.  

- Without it, container GUI apps get “Can’t open display” errors.  

### XEYES:	
Test app to confirm your Docker container can connect and display GUIs through X11 forwarding.  


## Conclusion  

- #### Built and ran a Docker container with Orange3 GUI.  

- #### Understood how to configure and test X11 forwarding using xeyes.  

- #### Learned to manage X server access with xhost.  

- #### Resolved common Docker+GUI pitfalls (permissions, module naming).   
  
  
## Orange3 SQL Table widget requires CSV:  

#### Orange3 SQL Table widget does NOT support SQLite files directly out of the box.  

#### Therefore, Convert SQLite DB to CSV and load CSV in Orange3:  
- Use sqlite3 CLI (once installed) to export the table as CSV
```	
	sqlite3 /app/workspace/data/gas.db
```

```
	sqlite> .tables
	sqlite> .headers on
	sqlite> .mode csv
	sqlite> .output table.csv
	sqlite> SELECT * FROM your_table_name;
	sqlite> .quit
```	

- You should find a file named table.csv

## Orange Datamining Lessons  
( YouTube tutorial - Orange Data Mining)

### Widgets

Widgets are computation units of Orange.
- read data  
- process it  
- visualize it  
- do clustering  
- build predictive model  
- help explore data  

Open data with File widget.

Widgets communicate with each other via input / output channels.

### Workflow  

Connecting a File widget with a Distribution wideget is a workflow.



