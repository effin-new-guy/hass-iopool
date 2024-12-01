# hass-iopool
[iopool](https://us.iopool.com/) integration with [Home Assistant](https://www.home-assistant.io/)

full credit to the friendly helpful folks that commented on this post
[https://github.com/iopool/community/discussions/14](url)

Additional credit to ReenigneArcher for the following post
[https://github.com/iopool/community/discussions/14#discussioncomment-6746728](url)

Which prompted me after struggling with yaml (as I constantly do) to document (and share) the steps I took to integrate my iopool EcO Smart Water Monitor with Home Assistant 

___

### Prerequisites:
Home Assistant including the following:
- installed and running
- using [Home Assistant Secrets](https://www.home-assistant.io/docs/configuration/secrets/) (for the api key)
- using [Home Assistant Pacakges](https://www.home-assistant.io/docs/configuration/packages/)
  
iopool EcO Smart Water Monitor including the following:
- working with the iopool cOnnect wi-Fi Gateway (you can see entries in the app indicating everything is working with your EcO)

___
  
## Steps to get iopool in Home Assistant:

in the iopool app go to the settings page and at the bottom create an API key. 
create the following entry in secrets.yaml
```yml
iopool_api_key: AIzaSyClzfrOzB818x55FASHvX4JuGQciR9lv7q
```
(that is not my API key, so it will not work for you,  you need to input your API key in the secrets.yaml)

in the pacakges directory in Home Assistant installation create an iopool.yaml file and place the following code

```yml
rest:
  - resource: "https://api.iopool.com/v1/pools"
    headers:
      x-api-key: !secret iopool_api_key
    sensor:
      - name: iopool_title
        icon: mdi:pool
        value_template: "{{ value_json[0].title }}"

      - name: iopool_mode
        icon: mdi:pool
        value_template: "{{ value_json[0].mode }}"

      - name: iopool_action_required
        icon: mdi:alert-box
        value_template: "{{ value_json[0].hasAnActionRequired }}"

      - name: iopool_temperature
        icon: mdi:pool-thermometer
        unit_of_measurement: '°F'
        value_template: "{{ (value_json[0].latestMeasure.temperature * 9 / 5 + 32) | int }}"  # degrees F
        # value_template: "{{'%0.1f' | format(value_json[0].latestMeasure.temperature) }}"  # degrees C

      - name: iopool_ph
        icon: mdi:ph
        unit_of_measurement: 'pH'
        value_template: "{{'%0.1f' | format(value_json[0].latestMeasure.ph) }}"

      - name: iopool_orp
        icon: mdi:scale-balance
        unit_of_measurement: 'mV'
        value_template: "{{ value_json[0].latestMeasure.orp }}"

      - name: iopool_filtration_duration_advice
        icon: mdi:pump
        unit_of_measurement: 'h'
        value_template: "{{ value_json[0].advice.filtrationDuration }}"
```

That is basically it.  
In Home Assistant, open Developer Tools, select "states" and type 'iopool' in the Entity and you should see multiple entries for the new sensors (screenshot below).

![alt text](https://github.com/effin-new-guy/hass-iopool/blob/main/images/iopool_.png)

To get all the cool dashboard stuff, go back to ReenigneArcher's post where he makes the modest claim of not being the best at setting up cards and blatantly copy his awesome cards and add them to your Home Assstant dashboard then thumbs up his post
[https://github.com/iopool/community/discussions/14#discussioncomment-6746728](url)

Conditional card:
```yml
card:
  content: >-
    <font color="orange"><center><h2>Pool requires attention,<br>check iopool
    app.</h2></center></font>
  type: markdown
conditions:
  - entity: sensor.iopool_action_required
    state: 'True'
type: conditional
view_layout:
  position: sidebar

Auto entities card:

type: custom:auto-entities
card:
  type: entities
filter:
  include:
    - name: iopool*
  exclude: []
```
Vertical stack in card water temp:
```yml
type: vertical-stack
cards:
  - graph: line
    type: sensor
    entity: sensor.iopool_temperature
    detail: 2
    name: Water temperature
    icon: ''
  - type: gauge
    entity: sensor.iopool_temperature
    segments:
      - from: 0
        color: '#db4437'
      - from: 59
        color: '#ffa600'
      - from: 69
        color: '#43a047'
      - from: 84
        color: '#ffa600'
      - from: 90
        color: '#db4437'
    min: 50
    max: 100
    needle: true
    name: Water Temperature
  - type: history-graph
    entities:
      - entity: sensor.iopool_temperature
```
Vertical stack in card ph:
```yml
type: vertical-stack
cards:
  - graph: line
    type: sensor
    entity: sensor.iopool_ph
    detail: 2
    icon: ''
    name: pH
  - type: gauge
    entity: sensor.iopool_ph
    segments:
      - from: 0
        color: '#db4437'
      - from: 6.8
        color: '#ffa600'
      - from: 7.1
        color: '#43a047'
      - from: 7.7
        color: '#ffa600'
      - from: 8.1
        color: '#db4437'
    min: 6.4
    max: 8.4
    needle: true
    name: pH
  - type: history-graph
    entities:
      - entity: sensor.iopool_ph
```
Vertical stack in card orp:
```yml
type: vertical-stack
cards:
  - graph: line
    type: sensor
    entity: sensor.iopool_orp
    detail: 2
    name: Disinfection potential
    icon: ''
  - type: gauge
    entity: sensor.iopool_orp
    segments:
      - from: 0
        color: '#db4437'
      - from: 550
        color: '#ffa600'
      - from: 650
        color: '#43a047'
      - from: 800
        color: '#ffa600'
      - from: 1000
        color: '#db4437'
    min: 350
    max: 1100
    needle: true
    name: Disinfection potential
  - type: history-graph
    entities:
      - entity: sensor.iopool_orp
```

