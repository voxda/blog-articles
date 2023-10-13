In this article we will explain how to measure the ultimate API uptime with Prometheus and Grafana to get 99.9% availability. In our [previous article](https://voxda.io/metrics-first-or-youll-suck-at-what-you-do/) we explained why IT metrics are important but very few shops get it right.  Usually IT Operations teams will generate a massive amount of dashboards of meaningless metrics that don't really give a good view of what's happening right now.  Support teams or Software Engineers will complain about environment stability but have no way to actually measure how available their software components are.  They will point to log error messages, postman results, and emotion to say things are bad.  Most IT teams suck at monitoring their applications and design overly complicated dashboards.  This leads to an ocean of URLs to be saved which nobody ever looks at. 

To skip to the tutorial: [Tutorial](###Tutorial)

### How do we fix this?
I propose a completely different model where a few great global views that produce a real time heat map of availability.  SREs would monitor these boards and react in real time and determine root cause.  As root cause is found a solution is put back into place immediately.  This cycle would repeat over a year or so until your system is rock solid.   Without this constant drive of identify and remediate, your systems will not get better.  

I'll show you how to measure an entire platform full of APIs or UIs in a single dashboard.  You will be able to spot an outage instantly and measure overall availability of an environment or even all environments in the SDLC.  This will be a powerful tool that makes all other dashboards just a supporting act for troubleshooting.

### Tools
Among the ocean of tools in the toolchain, [Prometheus](https://prometheus.io/) has risen as a powerhouse for monitoring and alerting toolkit initially developed at [SoundCloud](https://soundcloud.com/). When coupled with [Grafana](https://grafana.com/), a platform for monitoring and observability, it becomes a full-fledged powerhouse, enabling DevOps engineers to keep a close eye on their infrastructure and applications, including the Kubernetes clusters. In addition, we'll use the simple synthetic-monitor repo from [Voxda](https://github.com/voxda/synthetic-monitor) to generate custom Prometheus metrics used in this tutorial.  

#### Pros
1. **Comprehensive Monitoring:** Together, Prometheus and Grafana offer a comprehensive view of your systems with the ability to scrape metrics from various data sources.
2. **Community and Support:** Being open-source, both have vibrant communities that contribute to their development, offering a rich set of features and plugins.
3. **Highly Customizable:** They offer high customization potential, including the ability to develop your dashboards in Grafana, to tailor to your exact needs.

#### Cons
1. **Complex Setup:** Setting up Prometheus and Grafana can be a bit intricate, especially in a Kubernetes environment, often requiring a steep learning curve.  This can be overcome with help from ChatGPT and GitHub examples.
2. **Resource Intensive:** Depending on the scale of your operations, these tools can be resource-intensive, sometimes requiring significant computational resources.
3. **Potential Overhead:** For small-scale deployments or simpler setups, the combined utility of Prometheus and Grafana might introduce an unnecessary level of complexity and overhead.

### Tutorial - Ultimate API uptime with Prometheus and Grafana
In this tutorial we will setup an API site availability monitor to measure if we are hitting our 99.9% uptime goals. 

#### Step 1: Setting up Prometheus on your local machine
Let's clone some docker compose stacks for Prometheus and Grafana from the voxda repos.  
``` shell
git clone https://github.com/voxda/docker-compose-stacks.git
```
Next, let's use a terminal to get into the proper directory and run docker-compose.
``` shell
cd /<your-dir>/prometheus-grafana
docker-compose up
```
Now you can see grafana and prometheus running in your local browser
1. Prometheus: [http://localhost:9090](http://localhost:9090)
2. Grafana [http://localhost:3000](http://localhost:3000)

#### Step 2: Let's see what Prometheus is monitoring!
1. Select **Targets** from the Prometheus menu or click here: [http://localhost:9090/targets?search=](http://localhost:9090/targets?search=)
2. Click "Show More" on the blackbox section and there will be 3 endpoints being monitored. All three should have "State = Up" at this point.

#### Step 3: Add your own endpoints?
In your docker compose stack there is a module called Blackbox that was loaded to monitor 3 specific endpoints.  example.com, example.net, and example.org. **Blackbox** is an easy way to monitor an endpoint if you only need an up or down metric.  In future blog posts we will show you how to create your own custom metrics application using NodejS and the Prometheus client library.  
![Prometheus Targets](http://voxda.io/wp-content/uploads/2023/10/PrometheusTargetsBalckbox.png)

To add a new endpoint to Blackbox do the following:
1. Add scrape config job to the prometheus.yml file
``` shell
cd docker-compose-stacks/prometheus-grafana/prometheus
```  
2. Open the prometheus.yml and find the blackbox scrape configs
``` yaml
- job_name: 'blackbox'
    metrics_path: '/probe'
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - 'http://example.com'
        - 'http://example.org'
        - 'http://example.net'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox_exporter:9115
```
3. Under static_configs change the targets to anything you want to monitor. Note: many public sites have ways to detect bot or automated testing traffic. If your monitoring isn't up this is probably the reason.  
``` yaml
static_configs:
      - targets:
        - 'your endpoint'
```
4. Restart your docker compose stack.  You can do this in Docker Desktop using the stop and play buttons or you can use docker-compose down command. Once restarted check your blackbox target again.  You should see your custom endpoint with a "Status=Up"
  
#### Step 4: Let's create some cool stuff!
1. Navigate to Grafana or click here: [http://localhost:3000](http://localhost:3000).  The credentials for your local instance are admin/foobar.
2. Menu-->Dashboards-->New to create a new dashboard.  You can also create Folders to organize your dashboards or import dashboard json. 

![New Dashboard](http://voxda.io/wp-content/uploads/2023/09/GrafanaDashboardNew.png)
  
3. Select "Add Visualization" and the Prometheus as the data source.  

![Add Visualization](http://voxda.io/wp-content/uploads/2023/09/GrafanaDashboardNew2.png)

![Select Data](http://voxda.io/wp-content/uploads/2023/09/GrafanaDashboardNew3-SelectData.png)

4. Now you are presented with a complex query panel to design your visualization. On the surface this looks complex but as you learn it you realize it's very simple.  They just have lots of options to make your visuals look great.  Feel free to watch the corresponding youtube video for this section to make things easier.

![Query Panel](http://voxda.io/wp-content/uploads/2023/09/GrafanaDashboardNew4-QueryPanel.png)

5. Select State Timeline Visualization.  This will show our availability over time using a bar chart with green equals up and red equals down.  It's helpful to see availability patterns over time and this chart will tell the actual story of your application. 

![State Timeline](http://voxda.io/wp-content/uploads/2023/10/GrafanaDashboardNew-StateTimelineSelect.png)

6. Select or enter the following state timeline options.  These are the settings I prefer but experiment to get the look and feel you like.  
- Title: Availability over time
- Description: Shows availability over time
- Transparent background: On
- Merge consecutive values: false
- Show Values Never: true
- Row Height: 0.9
- Line Width: 0
- Fill Opacity: 100
- Connect null values: Never (this allows blank spots on the diagram which shows if your monitor is not working).  

![State Timeline Options](http://voxda.io/wp-content/uploads/2023/10/GrafanaDashboardNew-StateTimeline1.png)

7. Enter Tooltip and Standard Options. If not specified then accept the default value.  
- Tooltip: Single
- Unit: Number
- Color Scheme: Red-Yellow-Green (by value)

![State Timeline Tooltips](http://voxda.io/wp-content/uploads/2023/10/GrafanaDashboardNew-StateTimeline2.png)

8. Add value mappings. This is a critical step that maps a value to a color on your chart.  Red is the traditional color for issues and green typically means good but you could use any color scheme you want.   
- 1 = green
- 0 = red

![Value Mapping 1](http://voxda.io/wp-content/uploads/2023/10/GrafanaDashboardNew-StateTimeline-valuemap1.png)

![Value Mapping 0](http://voxda.io/wp-content/uploads/2023/10/GrafanaDashboardNew-StateTimeline-valuemap2.png)

9. Remove threshold default value of 80 and set base color to transparent.  This is not required for this chart but could be useful in other situations.  

![Remove Threshold 80](http://voxda.io/wp-content/uploads/2023/10/Grafana-removeThreshold.png)

![Set Base Color to Transparent](http://voxda.io/wp-content/uploads/2023/10/Grafana-removeThresholdBaseColor.png)

10. Enter the following query into the PromQL query section and change the legend to Custom and {{instance}}. Grafana allows you to enter code directly or use a visual query builder.  A good way to learn is to use ChatGPT to help you build queries and then switch to the visual query builder to see how it's built.  Over time you will get better at using the visual query builder and the PromQL syntax.  See the ChatGPT of this blog post for example prompts.   
```
probe_success{job="blackbox"}
```
![PrompQL](http://voxda.io/wp-content/uploads/2023/10/Grafana-queryCodePanel.png)

11. Your result should look like the following which shows availability over a time period.  Red sections mean your endpoints are down and green means your endpoints are up.  This is an excellent view when performing chaos monkey testing or trying to find outage patterns.  

![State Timeline Green Example](http://voxda.io/wp-content/uploads/2023/10/Grafana-stateTimeLineGreenExample.png) 

#### Step 5: Let's calculate 99.9% Availability
1. Add a new visualization and select Gauge as the visualization type.

![Select Gauge](http://voxda.io/wp-content/uploads/2023/09/GrafanaDashboardNew5-GaugeSelect.png)

2. Enter the following PrompQL code and update the Legend to Custom and {{instance}}. The following query code performs an average over the selected time range and displays as a percentage.  You could get as complex with this query as you want but this simple one works well.  
```
avg_over_time(probe_success{job="blackbox"}[$__range])*100
```

![PrompQL for three 9's](http://voxda.io/wp-content/uploads/2023/10/Grafana-promQlForThree9s.png)
     
3. Update the Gauge Standard Options with the following settings.
- Unit: Percent (0-100)
- Decimals: 1
- Color Scheme: From Thresholds (by value)

![Gauge Standard Options](http://voxda.io/wp-content/uploads/2023/10/Grafana-Guage-StandardOptionsThree9s.png)

4. Change the Thresholds to reflect our red, yellow, green values
- Green: 99.9
- Yellow: 90.0
- Red: 0
- Base: Transparent

![Ultimate API uptime with Prometheus and Grafana](http://voxda.io/wp-content/uploads/2023/10/Grafana-Gauge-ThresholdsThree9s.png)

5. Press Apply and resize each panel to the size you prefer by dragging the bottom right hand corner.  

#### Final Result - Ultimate API uptime with Prometheus and Grafana
Your dashboard look like the following!  If not check your settings and adjust until you get the desired effect.  
![Ultimate API uptime with Prometheus and Grafana](http://voxda.io/wp-content/uploads/2023/10/Grafana-Dashboard-Three9s.png)

### Conclusion
In the ever-evolving landscape of DevOps, the duo of Prometheus and Grafana might initially come off as a pain point given their complex setup and resource demands. However, once past the initial hurdles, they unfold as highly beneficial tools offering deep insights into your system's health and performance, thus solidifying their place in the toolkit of any serious DevOps engineer.

See below for some additional information on helpful ChatGPT prompts, Learning Videos, Related Courses, and some random cool stuff to buy.  

### ChatGPT - Ultimate API uptime with Prometheus and Grafana
This section will provide a few ChatGPT prompts to make working with Grafana and Prometheus easier. Note: models change frequently and I cannot guarantee success when using ChatGPT.  Verify results before using.  

1. Calculate three 9s availability for blackbox exporter metrics.  
```
You are an expert in Grafana and Prometheus and creating PromQL for advanced Grafana Dashboards.  Provide a query against the blackbox probe_success metrics that calculates 99.9% availability using the selected dashboard time range.
```

2. Explain how to create a state timeline
```
You are an expert in Grafana and Prometheus and creating PromQL for advanced Grafana Dashboards. Explain how to create a Grafana State Timeline to show availability of an endpoint over a selected time range. 
```

3. Explain how to use Blackbox Exporter
```
You are an expert in Grafana and Prometheus and creating PromQL for advanced Grafana Dashboards. Explain how to setup and use blackbox exporter for prometheus on a windows laptop.  
```
