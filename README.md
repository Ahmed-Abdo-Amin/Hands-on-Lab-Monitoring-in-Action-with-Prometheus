# Hands-on Lab: Monitoring in Action with Prometheus
<style>H1{color:Blue;}</style>
## <img src="./images/intr.png" width='60'> Introduction
Welcome to the Monitoring with Prometheus lab. In this lab, you will become familiar with using Prometheus to monitor sample servers simulated with node exporter. You will use Prometheus to monitor the target node_exporter application that is configured by scraping metrics endpoints of the node_exporter. You will finish the lab by learning how to instrument a Python Flask application to emit metrics and deploy that application so that Prometheus can monitor it.

## <img src="./images/objectives.png" width='60'> Learning Objectives

After completing this exercise, you should be able to:
- Configure the targets for Prometheus to monitor
- Create queries to get the metrics about the target
- Determine the status of the targets
- Identify information about the targets and visualize it with graphs
- Instrument a Python Flask application to be monitored by Prometheus

## Prerequisites


This lab uses Docker to run both Prometheus, and special Node Exporters, which will behave like servers that you can monitor. As a prerequisite, you will pull down the <span style="color: red;">bitnami/prometheus:latest</span> image and the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">bitnami/node-exporter</span> image from Docker Hub. You will use these images to run Prometheus and create three instances of node exporters to be monitored.

## The Task
1. To start this lab, you will need a terminal. If a terminal is not already open, you can open one from the top menu. Go to Terminal and choose New Terminal to open a new terminal window.
![new_terminal](./images/new_terminal.png)

2. Next, use the following <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">docker pull</span> command to pull down the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">bitnami/node-exporter</span> image from Docker Hub that you will use to simulate three servers being monitored.
>>> docker pull bitnami/node-exporter:latest

Your output should look similar to this:
![prom_pull_prometheus](./images/prom_pull_node_exporter.png)

3. Then, pull the Prometheus docker image into your lab environment, by running the following <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">docker pull</span> command in the terminal.

>>> docker pull bitnami/prometheus:latest

Your output should look similar to this:
![prom_pull_prometheus](./images/prom_pull_prometheus.png)

## <img src="./images/step-1.png" width='60'> Start the first node exporter:
The first thing you will need is some server nodes to monitor. You will start up three node exporters listening on port  <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9100</span> and forwarding to ports  <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9101</span>,  <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9102</span>, and  <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9103</span>, respectively. Each node will need to be started up individually.

In this step, you will create a Docker network for all of the node exporters and Prometheus to communicate on, and start just the first node, <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9101</span>, and ensure it is working correctly.

## The Task
1. Start by running the following <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">docker network</span> command to create a network called <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">monitor</span> within which we will run all of the docker containers.

>>> docker network create monitor

Your output should look similar to this:
![create-monitor](./images/create-monitor.png)

2. Next, run the following <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">docker run</span> command to start a node exporter instance on the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">monitor</span> network, listening at port <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9101</span> externally and forwarding to port <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9100</span> internally.
>>> docker run -d --name node-exporter1 -p 9101:9100 --network monitor bitnami/node-exporter:latest

This will start an instance named node_exporter1 of node-exporter. The output should look something like this (note: the container id will be different each time):
![prom_run_node_exporter](./images/prom_run_node_exporter.png)

3. Next, check if the instance is running by pressing the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">[Launch Application]</span> link, which will launch the application on port <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9101</span>:

[Launch Application](https://ahmedabdoami-9101.theiaopenshift-0-labs-prod-theiaopenshift-4-tor01.proxy.cognitiveclass.ai/)

4. You should see the Node Exporter page open up with a hyperlink to Metrics. These are the metrics the Prometheus instance is going to monitor.
![node_exporter](./images/node_exporter_light.png)

5. Finally, click the Metrics link to see the metrics.
![metrics_view_light](./images/metrics_view_light.png)

## <img src="./images/step-2.png" width='60'> Start two more node exporters:
Now that you have one node exporter working, you can start two more so that Prometheus has three nodes to monitor in total. You will do this the same way as you did the first node exporter, except that you will change the external port numbers to <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9102</span> and <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9103</span>, respectively.

## The Task
1. In the terminal, run the following commands to start two more instances of node exporter.

>>> docker run -d --name node-exporter2 -p 9102:9100 --network monitor bitnami/node-exporter:latest

![Run-first-node-exporter](./images/Run-first-node-exporter.png)
and
>>> docker run -d --name node-exporter3 -p 9103:9100 --network monitor bitnami/node-exporter:latest

![](./images/Run-Other-nodes-exporter.png)

2. Now, check if all the instances of node exporter are running by using the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">docker ps</span> command and pipe it through the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">grep</span> command to search for <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">node-exporter</span>.

>>> docker ps | grep node-exporter

## Results
If everything started correctly, you should see output similar to the following coming back from the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">docker ps</span> command:
![prom_docker_ps](./images/prom_docker_ps.png)
You are now ready to configure and run Prometheus.

## <img src="./images/step-3.png" width='60'> Configure and run Prometheus:
Before you can start Prometheus, you need to create a configuration file called <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">prometheus.yml</span> to instruct Prometheus on which nodes to monitor.

In this step, you will create a custom configuration file to monitor the three node exporters running internally on the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">monitor</span> network at <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">node-exporter1:9100</span>, <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">node-exporter2:9100</span>, and <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">node-exporter3:9100, respectively</span>. Then you will start Prometheus by passing it the configuration file to use.

## The Task
1. First, use the touch command to create a file named <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">prometheus.yml</span> in the current directory. This is the file where you will configure the Prometheus to monitor the node exporter instances.
>>> touch /home/project/prometheus.yml

2. Next, from Explorer, navigate to Project, and then select prometheus.yml to edit the file.

3. Then, copy and paste the following configuration contents into the yaml file and save it:

        # my global config
        global:
        scrape_interval: 15s # Set the scrape interval to every 15 seconds. The default is every 1 minute.

        scrape_configs:
        - job_name: 'node'
            static_configs:
            - targets: ['node-exporter1:9100']
                labels:
                group: 'monitoring_node_ex1'
            - targets: ['node-exporter2:9100']
                labels:
                group: 'monitoring_node_ex2'
            - targets: ['node-exporter3:9100']
                labels:
                group: 'monitoring_node_ex3'

Notice that while you access the node exporters externally on ports <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9101</span>, <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9102</span>, and <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9103</span>, they are internally all listening on port <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">9100</span>, which is how Prometheus will communicate them on the monitor network.

### Take a look at what this file is doing:

- Globally, you set the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">scrape_interval</span> to 15 seconds instead of the default of 1 minute. This is so that we can see results quicker during the lab, but the 1 minute interval is better for production use.
- The <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">scrape_config</span> section contains all the jobs that Prometheus is going to monitor. These job names have to be unique. You currently have one job called <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">node</span>. Later we will add another to monitor a Python application.
- Within each job, there is a <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">static_configs</span> section where you define the targets and define labels for easy identification and analysis. These will show up in the Prometheus UI under the Targets tab.
- The targets you enter here point to the base URL of the service running on each of the nodes. Prometheus will add the suffix <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">/metrics</span> and call that endpoint to collect the data to monitor from. (For example, <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">node-exporter1:9100/metrics</span>)

You will have an opportunity to create your own Prometheus file to monitor your Python application in the practice exercise.

4. Finally, you can launch the Prometheus monitor in the terminal by executing the following <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">docker run</span> command passing the yaml configuration file as a volume mount with the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">-v</span> parameter.

>>> docker run -d --name prometheus -p 9090:9090 --network monitor \
-v $(pwd)/prometheus.yml:/opt/bitnami/prometheus/conf/prometheus.yml \
bitnami/prometheus:latest

Note: This Dockerized distribution of Prometheus from Bitnami expects its configuration file to be in the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">/opt/bitnami/prometheus/conf/prometheus.yml</span> file, which is why you are mapping your <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">prometheus.yml</span> file to this location. Other distributions may look in other locations. Always check the documentation to be sure of where to mount the configuration file.

## Results
You should see just the Prometheus container id returned, indicating that Docker has started Prometheus in the background.
![prom_run_prometheus](./images/prom_run_prometheus.png)
You are now ready to do some monitoring.

## <img src="./images/step4.png" width='60'>Open the Prometheus UI:

In this step, you will launch the Prometheus web UI in an external browser window and navigate to the page where you start executing queries.

1. Open the Prometheus web UI by clicking Skills Network Toolbox. Under Other, select Launch Application, in Application Port enter the port number 9090, and then click the launch URL button.

![launch_9090](./images/launch_9090.png)

2. The Prometheus application UI opens up by default in the graph endpoint.

![prometheus_ui_light](./images/prometheus_ui_light.png)

3. Next, in the Prometheus application, click Status on the menu and choose Targets to see which targets are being monitored.
![prometheus_status_light](./images/prometheus_status_light.png)

4. View the status of all three node exporters.
![targets_up](./images/targets_up.png)

5. Click Graph to return to the home page.
![graphs](./images/graphs.png)
You are now ready to execute queries.

## <img src="./images/step-5.png" width='60'> Execute your first query:

You are now ready to execute your first query. The first query you run will query the nodes for the total CPU seconds. It will show the graph as given in the image. You can observe the details for each instance by hovering the mouse over that instance.

## The Task
1. Ensure you are on the Graph tab, and then copy-n-paste the following query and press the blue Execute button on the right or press return on your keyboard to run it. It will show the graph as given in the image. You can observe the details for each instance by hovering the mouse over that instance.
>>> node_cpu_seconds_total

![node_cpu_graph](./images/node_cpu_graph.png)

2. Next, click Table to see the CPU seconds for all the targets in tabular format.

![node_cpu_table](./images/node_cpu_table.png)

3. Now, filter the query to get the details for only one instance <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">node-exporter2</span> using the following query.

>>> node_cpu_seconds_total{instance="node-exporter2:9100"}

![node_instance](./images/node_instance.png)

4. Finally, query for the connections each node has using this query.

>>> node_ipvs_connections_total

![node_connections_light](./images/node_connections_light.png)

## <img src="./images/step-6.png" width='60'> Stop and observe:
In this step, we will stop one of the node exporter instances and see how that is reflected in the Prometheus console.

## The Task
1. Stop the node-exporter1 instance by running the following <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">docker stop</span> command and then switch back to the old terminal in which Prometheus is running.
>>> docker stop node-exporter1

2. Now go back to the Prometheus UI on your browser and check the targets by selecting the menu item Status -> Targets.

## Results
You should now see that one of the node exporters that are being monitored is down. The nodes might not be displayed in the same order, but the node which is should be <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">node-exporter1</span>, the node that you stopped.

Note: You configured Prometheus to scrape every 15 seconds, so you may have to wait that long and press refresh on your browser to see the status change.

![node_down](./images/node_down.png)

## <img src="./images/Step-7.png" width='60'> Enable your application: 

Monitoring node exporters is fine for a demonstration, but you are a software engineer. You need to know how to enable your applications to be monitored by Prometheus. There is no magic here. Metrics do not simply appear out of nowhere. You must instrument your application to emit metrics on an endpoint called <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">/metrics</span> in order for Prometheus to be able to monitor your application.

Luckily there is a Python package called Prometheus Flask exporter for Prometheus that will do this for you. In this step, you will create a simple Python Flask application and enable a metrics endpoint so that you can monitor it.

## The Task

Below is a code for a Python Flask server with three end points, <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">/</span>, <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">/home</span>, and <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">/contact</span>. The code uses the package <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">prometheus_flask_exporter</span> for generating metrics for Prometheus to monitor.

1. First, create a file named <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">pythonserver.py</span> in the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">/home/project</span> folder. Press the button below and answer Create the file when prompted.

![prom_create_file](./images/prom_create_file.png)

2. Then, paste the following code content into it:

        from prometheus_flask_exporter import PrometheusMetrics
        from flask import Flask

        app = Flask(__name__)
        metrics = PrometheusMetrics.for_app_factory()
        metrics.init_app(app)

        @app.route('/')
        def root():
            return 'Hello from root!'

        @app.route('/home')
        def home():
            return 'Hello from home!'

        @app.route('/contact')
        def contact():
            return 'Contact us!'

        if __name__ == '__main__':
            app.run(host="0.0.0.0", port=8080)

Notice that you only had to import the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">PrometheusMetrics</span> class from the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">prometheus_flask_exporter</span> package and add two lines of code to instantiate a <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">PrometheusMetrics.for_app_factory()</span> as metrics, and call <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">metrics.init_app(app)</span> to initialize it. That is it! Three total lines of code, and you have Prometheus support!

3. Next, you need to deploy this code on the same docker network as Prometheus. To do this, create a file named <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">Dockerfile</span> in the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">/home/project</span> folder:

4. Paste the following contents into Dockerfile and save it:

        FROM python:3.9-slim
        RUN pip install Flask prometheus-flask-exporter
        WORKDIR /app
        COPY pythonserver.py .
        EXPOSE 8080
        CMD ["python", "pythonserver.py"]

5. Now, use the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">docker build</span> command to build a Docker image for the service (Note: You can safely ignore any red output from the docker build command. It just warns about running pip as root):

>>> docker build -t pythonserver .

6. Finally, run the pythonserver Docker container on the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">monitor</span> network exposing port 8080 so that Prometheus will have access to it:

>>> docker run -d --name pythonserver -p 8081:8080 --network monitor pythonserver

7. (Optional) Check that the Python server is running by pressing the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">Launch Python Server UI</span> link:

>>> [Launch Python Server UI](https://ahmedabdoami-8081.theiaopenshift-0-labs-prod-theiaopenshift-4-tor01.proxy.cognitiveclass.ai/)

You are now ready to add your new application to Prometheus.

## <img src="./images/step8.png" width='60'> Reconfigure Prometheus:
Now that you have your application running, it is time to reconfigure Prometheus so that it knows about the new <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">pythonserver</span> node to monitor. You can do this by adding the Python server as a target in your <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">prometheus.yml</span> file.

## The Task
1. First, open the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">prometheus.yml</span> file:

2. Next, create a new job to monitor the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">pythonserver</span> service that is listening on port <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">8080</span>. Use the previous job as an example.

Create a new job name and change the target in prometheus.yml to point to the server url of your `pythonserver` and port:

    - job_name: {make up a job name here}
    static_configs:
        - targets: [{place the target to monitor here}]
        labels:
            group: {make up a group name here}

You may have created a different `job_name` or `group` but the `targets` must match the target below:

    - job_name: 'monitorPythonserver'
    static_configs:
        - targets: ['pythonserver:8080']
        labels:
            group: 'monitoring_python'

3. Check that your complete <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">prometheus.yml</span> file looks similar to this one:

        # my global config
        global:
        scrape_interval: 15s # Set the scrape interval to every 15 seconds. The default is every 1 minute.

        scrape_configs:
        - job_name: 'monitorPythonserver'
            static_configs:
            - targets: ['pythonserver:8080']
                labels:
                group: 'monitoring_python'

        - job_name: 'node'
            static_configs:
            - targets: ['node-exporter1:9100']
                labels:
                group: 'monitoring_node_ex1'
            - targets: ['node-exporter2:9100']
                labels:
                group: 'monitoring_node_ex2'
            - targets: ['node-exporter3:9100']
                labels:
                group: 'monitoring_node_ex3'

4. Restart the prometheus server to pick up the new configuration changes:

>>> docker restart prometheus

5. Check the Prometheus UI to see the new Targets.

Note: You may have to click the “show more” button next to <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">monitorPythonserver</span> as depicted below:

![prom_targets_show_more](./images/prom_targets_show_more.png)

## Results

If everything went well, when you open the Prometheus targets, you will see the status of your Python server as in the image below.

![prom_python_server](./images/prom_python_server.png)

## <img src="./images/step9.png" width='60'> Monitor your application:

In order to see some results of monitoring, you need to generate some network traffic.

1. Make multiple requests to the three endpoints of the Python server you  reated in the previous task and observe these calls on Prometheus.


        curl localhost:8081
        curl localhost:8081/home
        curl localhost:8081/contact


>>> Feel free to run these multiple times to simulate real network traffic.

2. Use the Prometheus UI to query for the following metrics.

        flask_http_request_duration_seconds_bucket
        flask_http_request_total
        process_virtual_memory_bytes

If you are interested in what other metrics are being emitted by your application, you can view all of the metrics that your application is emitting by opening the <span style="color:orange; 
        border-style: solid; 
        border-width: thin; 
        border-color:black">/metrics</span> endpoint just like Prometheus does. Feel free to experiment by running queries against other metrics:

![prom_app_metrics](./images/prom_app_metrics.png)
