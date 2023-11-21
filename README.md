# Project Description

## Introduction:
The Three phases Energy analyse and monitor is project that aims to accurately measure and present the digital twin representation of power quantity and quality in three-phase systems. This system leverages the ADE9000 integrated circuit for precise measurements. The data is then transmitted via the ISP interface to an ESP8266, which communicates over Wi-Fi with a FIWARE context broker. The FIWARE context broker is responsible for data storage and communication with Grafana for visualization.

## Project Objectives:
L'objectif central de notre projet est de développer un système de smart metering qui permettra la mesure précise et complète des valeurs quantitatives et qualitatives associées aux systèmes de production et de consommation d'énergie électrique. Contrairement à la focalisation sur le ADE 9000, notre attention est portée sur la création d'un outil innovant et performant qui répondra aux besoins spécifiques liés à la surveillance et à l'optimisation des processus énergétiques.

## Systèmes monitorés:

Le système axé sur la mesure et la surveillance de l'énergie en trois phases, pourrait être utilisé pour surveiller divers systèmes de production et de consommation d'énergie électrique. Voici quelques exemples de systèmes que le système peut monitorer :


1. Réseau Électrique Local : Le système pourrait être utilisé pour surveiller la consommation d'énergie dans un bâtiment, permettant aux utilisateurs de prendre des décisions informées pour économiser de l'énergie.
Le système pourrait surveiller la consommation d'énergie pour l'éclairage, le chauffage, la climatisation, etc., afin d'optimiser l'efficacité énergétique.

2. Systèmes d'Énergie Renouvelable : Il pourrait être utilisé pour surveiller les systèmes de production d'énergie renouvelable tels que les parcs éoliens, les centrales solaires, etc.

# Architecture

![image](https://github.com/FiwareAtSupCom/3P_nrj_monitor/assets/93084127/8d696b53-4b43-4abd-801d-2c5109747c3d)

# Système embarqué

L'ESP32 effectue des mesures périodiques via des interruptions de timer. Les données sont transmises par Wi-Fi, et en cas d'interruption de la connexion, l'ESP32 stocke les informations dans une mémoire limitée de 512 octets. Elle surveille activement la connexion Wi-Fi, ajustant les timers pour minimiser le stockage local si la connexion est interrompue. Cette approche garantit une gestion efficace des données malgré les perturbations de la connectivité
![My Image](img/processes_sur_ESP-32.jpg width="40" height="400")
![My Image](img/power_interrpt.jpg width="40" height="400")
![My Image](img/Energy_interupt.jpg width="40" height="400")
## Schema
...
## code
...
# NGSI / datamodeles 
The necessary configuration information can be seen in the services section of the associated docker-compose.yml file:

```yaml
orion:
    image: quay.io/fiware/orion:latest
    hostname: orion
    container_name: orion
    depends_on:
        - mongo-db
    networks:
        - default
    expose:
        - '1026'
    ports:
        - '1026:1026'
    command: -dbhost mongo-db -logLevel DEBUG
  ```

```yaml
  mongo-db:
    image: mongo:4.2
    hostname: mongo-db
    container_name: db-mongo
    expose:
        - '27017'
    ports:
        - '27017:27017'
    networks:
        - default
  ```

The data we will use is:


• activeEnergyImport: Imported active energy consumed per phase since the counting start date.

• activePower: Active power consumed per phase since the counting start date.

• current: 'Electric current.

• dateEnergyMeteringStarted: start date of energy metering.

• dateModified: Timestamp of the last modification of the entity.


• Power factor :


The requests for writing IoT systems corresponding to the NGSI are found in the following files: 

https://github.com/FiwareAtSupCom/3P_nrj_monitor/blob/main/REQUETE%203%20phases.txt 

https://github.com/FiwareAtSupCom/3P_nrj_monitor/blob/main/requete%20solar.txt

the common structure within each data entity must be standardized to promote reuse.
The data model can be found in the following file: https://github.com/FiwareAtSupCom/3P_nrj_monitor/blob/main/data-model.txt

The digital twin would be constantly updated in real time using data from the ESP. It would reflect fluctuations in power consumed. To enable easy interaction, the digital twin would have a graphical user interface that would provide intuitive visualizations and performance charts.
# Base de données et registers
## Context Broker (Orion)

The FIWARE Orion Context Broker serves as the central hub for managing context data in our IoT system. It receives requests using the NGSI-v2 standard and handles crucial aspects such as data entities, subscriptions, and registrations.

### MongoDB

MongoDB is employed as the backend for the Orion Context Broker. It plays a pivotal role in storing context data information, managing data entities, and facilitating various operations related to context management.

## QuantumLeap

FIWARE QuantumLeap is a powerful generic enabler designed to simplify the storage and retrieval of time-series data generated by the Orion Context Broker. Unlike the STH-Comet generic enabler, QuantumLeap seamlessly integrates with time-series databases, offering compatibility with databases like CrateDB and TimescaleDB.

## CrateDB

CrateDB serves as a dedicated data sink for our IoT system, with a specific focus on handling time-based historical context data. As a distributed SQL database system tailored for IoT applications, CrateDB excels in ingesting a high volume of data points per second and supports real-time querying. Its capabilities make it well-suited for handling complex queries involving geospatial and time-series data.
### Connecting FIWARE to CrateDB via QuantumLeap

In the configuration, QuantumLeap listens to NGSI v2 notifications on port 8668 and persists historic context data to CrateDB. CrateDB is accessible using port 4200 and can either be queried directly or attached to the Grafana analytics tool.

### CrateDB Database Server Configuration

To set up the CrateDB Database Server, use the following configuration in your docker-compose.yml file:

```yaml
crate-db:
    image: crate:4.1.4
    hostname: crate-db
    ports:
        - '4200:4200'
        - '4300:4300'
    command:
        crate -Clicense.enterprise=false -Cauth.host_based.enabled=false  -Ccluster.name=democluster
        -Chttp.cors.enabled=true -Chttp.cors.allow-origin="*"
    environment:
        - CRATE_HEAP_SIZE=2g
```
### QuantumLeap Configuration
```yaml
  quantumleap:
    image: smartsdk/quantumleap
    hostname: quantumleap
    ports:
        - '8668:8668'
    depends_on:
        - crate-db
    environment:
        - CRATE_HOST=crate-db
    networks:
      - fiware_network
```
## Front End / Grafana ?
The choice of technology to use:
![image1](https://github.com/FiwareAtSupCom/3P_nrj_monitor/assets/93084127/58f27566-6620-4446-9689-2a541a62fe44)

• Ease of Use and Flexibility: Grafana provides a user-friendly interface for creating visualizations and dashboards without extensive coding. Its flexibility allows easy integration with various data sources, including CrateDB and Fiware solutions, making it a versatile choice.

• Real-Time Data Visualization: Grafana supports real-time data visualization, allowing you to create live dashboards that update dynamically as new data comes in. This is crucial for monitoring grid systems that require up-to-date information.

• Large Variety of Visualization Options: Grafana offers a wide array of visualization options. This variety enables you to choose the most suitable visualization type for different types of grid system data.

• Alerting and Monitoring Capabilities: Grafana provides alerting functionalities that can be set up based on the data from CrateDB and Fiware, allowing for proactive monitoring of the grid systems. This ensures timely responses to any anomalies or critical situations.

• Community Support and Integration: Grafana has a large community of users and developers, resulting in extensive documentation, plugins, and support. It integrates well with various data sources, making it easier to combine data from CrateDB and Fiware in a unified dashboard.

• Using a native frontend or other solutions might be more time-consuming and may lack the robustness and features that Grafana offers specifically for data visualization and monitoring. 
![image2](https://github.com/FiwareAtSupCom/3P_nrj_monitor/assets/93084127/11e5ede7-3e0e-46d0-bb30-d42cb9653795)

Grafana, in this context, appears to be a strong choice due to its ease of use, real-time capabilities, extensive visualization options, and community support, allowing you to focus more on the representation and analysis of the data rather than the complexities of building a visualization platform from scratch.




