# Card-mod customization
For full reference and documents go to the [card-mod repo](https://github.com/thomasloven/lovelace-card-mod), and find a [huge community thread](https://community.home-assistant.io/t/card-mod-add-css-styles-to-any-lovelace-card/120744) and a dedicated [topic on its powerful theming](https://community.home-assistant.io/t/card-mod-super-charge-your-themes/212176) options.

Below a short summary of many templates we use in [custom-ui](https://github.com/Mariusthvdb/custom-ui), but can also be replaced by these card-mod modifications.
Be aware that these card-mod modifications only show in the primary Frontend interface/Dashboard, and do not show in the secondary panels like more-info or other dialogs. If you need that, your only option is custom-ui. 
In all honestly, we cant promise that won't break in the (near) future beyond repair.

-----
Set an individual filter on an entity entity_picture
```
entity: person.mariusthvdb
card_mod:
  style:
    hui-generic-entity-row:
      $: |
        state-badge {
          filter: {{'grayscale(100%)' if not is_state(config.entity,'home')
                    else 'none'}};
        }
```
Set an entity_picture in stead of an icon and hide the icon:
```
entity: input_select.intercom_language
card_mod:
  style:
    hui-generic-entity-row:
      $: |
        state-badge {
          {% set flag = states(config.entity)|lower %}
          background-image: url("/local/flags/{{flag}}.png");
        }
    .: |
      :host {
        --card-mod-icon: none;
      }
```

Set an individual entity icon_color
```
entity: sensor.airpollution_level
card_mod:
  style: |
    :host {
      --paper-item-icon-color:
        {% set state = states(config.entity) %}
        {% set indexColor =
          {'good':'seagreen','moderate':'gold',
           'unhealthy_sensitive':'orange','unhealthy':'crimson',
           'very_unhealthy':'purple','hazardous':'maroon'} %}
         {{indexColor.get(state,'grey')}};
```

Set an individual entity icon
```
entity: sensor.wifi_signal_strength
card_mod:
  style: |
    :host {
      --card-mod-icon:
        {% set signal = states(config.entity)|int(default=0) %}
        {% if signal == 'unknown' %} mdi:help
        {% elif signal >= -50 %} mdi:wifi-strength-4
        {% elif signal >= -60 %} mdi:wifi-strength-3
        {% elif signal >= -67 %} mdi:wifi-strength-2
        {% elif signal >= -70 %} mdi:wifi-strength-1
        {% elif signal >= -80 %} mdi:wifi-strength-outline
        {% else %} mdi:wifi-strength-alert-outline
        {% endif %};
```

Set both icon and icon_color:
```
entity: input_select.tts_gender
card_mod:
  style: |
    :host {
      --card-mod-icon:
          {% set gen = states(config.entity) %}
          {{'mdi:human-female' if gen == 'Female' else 'mdi:human-male'}};
      --paper-item-icon-color:
          {% set gen = states(config.entity) %}
          {{'maroon' if gen == 'Female' else 'dodgerblue'}};
    }

entity: sensor.zigbee_connectivity
card_mod:
  style: |
    :host {
      --card-mod-icon:
        {% set con = states(config.entity) %}
        {{'mdi:zigbee' if con == 'connected' else 'mdi:wifi-strength-alert-outline'}};
      --paper-item-icon-color:
        {{'green' if con == 'connected' else 'maroon'}};
    }

entity: input_number.intercom_volume
card_mod:
  style: |
    :host {
      --card-mod-icon:
        {% set vol = states(config.entity)|float(0) %}
        {% if vol == 0.0 %} mdi:volume-off
        {% elif vol <= 0.3 %} mdi:volume-low
        {% elif vol <= 0.6  %} mdi:volume-medium
        {% else %} mdi:volume-high
        {% endif %};
      --paper-item-icon-color:
        {% if vol == 0.0 %} gray
        {% elif vol <= 0.3 %} dodgerblue
        {% elif vol <= 0.6 %} green
        {% elif vol <= 0.8 %} orange
        {% else %} red
        {% endif %};
    }

# note you only need to set the initial variable `{% set con = states(config.entity) %}` once,
# and it is used in both mods!
```
Set an icon_color in an [`custom:auto-entities`](https://github.com/thomasloven/lovelace-auto-entities) card:
```
- type: custom:auto-entities
  card:
    type: entities
    card_mod: &style
      style: |
        ha-card {
          box-shadow: none;
          margin: 0px -16px -16px -16px;
        }
  show_empty: false
  filter:
    include:
      - entity_id: '*air_quality_*'
        options:
          card_mod:
            style: |
              :host {
                --paper-item-icon-color:
                  {% set state = states(config.entity)|int(default=-5) %}
                  {% if state == 'unknown' %} gray
                  {% elif state <= 50 %} seagreen
                  {% elif state <= 100 %} gold
                  {% elif state <= 150 %} orange
                  {% elif state <= 50 %} crimson
                  {% elif state <= 100 %} purple
                  {% elif state <= 150 %} maroon
                  {% else %} red
                  {% endif %}
      - entity_id: '*air_pollution_level'
        options:
          card_mod:
            style: |
              :host {
                --paper-item-icon-color:
                  {% set state = states(config.entity) %}
                  {% set indexColor =
                    {'good':'seagreen','moderate':'gold',
                     'unhealthy_sensitive':'orange','unhealthy':'crimson',
                     'very_unhealthy':'purple','hazardous':'maroon'} %}
                   {{indexColor.get(state,'grey')}};
              }
```

Set an icon-color on a [`custom:template-entity-row`](https://github.com/thomasloven/lovelace-template-entity-row) (which already supports icon templates, but needs the mod for a color). Uses a different DOM-path,
so posting here as a further reference:
```
- type: custom:template-entity-row
  entity: input_number.alarm_volume
  state: >
    {{states(config.entity)|float(0)}} %
  icon: >
    {% set vol = states(config.entity)|float(0) %}
    {% if vol == 0.0 %} mdi:volume-off
    {% elif vol <= 0.3 %} mdi:volume-low
    {% elif vol <= 0.6  %} mdi:volume-medium
    {% else %} mdi:volume-high
    {% endif %}
  card_mod:
    style:
      div#wrapper:
        state-badge $: |
          ha-state-icon {
            color:
              {% set vol = states(config.entity)|float(0) %}
              {% if vol == 0.0 %} gray
              {% elif vol <= 0.3 %} dodgerblue
              {% elif vol <= 0.6 %} green
              {% elif vol <= 0.8 %} orange
              {% else %} red
              {% endif %}
          }
```
## Re-use a mod in the same file

We can use yaml anchors to copy&paste these mods:

```
entities:
  - entity: person.mariusthvdb
    <<: &person_picture
    card_mod:
      style:
        hui-generic-entity-row:
          $: |
            state-badge {
              filter: {{'grayscale(100%)' if not is_state(config.entity,'home')
                        else 'none'}};
            }
  - entity: person.mylovely
    <<: *person_picture
```

As always with Yaml anchors this is a *per file* option (can't c&p across the full config), and only complete yaml blocks can be pasted. Still, some creativity is possible, check the second `no_icon` on the second input_select entity:

```
- entity: input_select.mode
  card_mod:
    style:
      hui-generic-entity-row:
        $: |
          state-badge {
            {% set state = states(config.entity)|slugify %}
            {% set path = '/local/modes/' %}
            background-image:
              url('{{path}}{{state}}.png');
          }
      <<: &no_icon
        .: |
          :host {
            --card-mod-icon: none;
          }
- entity: input_select.activiteit
  card_mod:
    style:
      hui-generic-entity-row:
        $: |
          state-badge {
            {% set state = states(config.entity)|slugify %}
            {% set path = '/local/activities/' %}
            background-image:
              url('{{path}}{{state}}.png');
          }
      <<: *no_icon
```
We won't go into the yaml anchors any deeper here, please find some suggested reading in [Thomas Loven's publication](http://thomasloven.com/blog/2018/08/YAML-For-Nonprogrammers/) and .

## Re-use mods system wide
----
To have  **globally available** card-mod customizations, save these inside your `secrets.yaml` and insert them via
`!secret <secret>` tag. **(NOTE: this is YAML mode only, and not supported in the UI-mode and editor.)**

Example:
```
entity: person.mariusthvdb:
card_mod: !secret not_home_picture
```
-----
See below for a collection of generic mods, used throughout the config, that only have to be written once, and can be imported by that tiny `!secret` reference:

```
##########################################################################################
# Card mod stylings frequently used saved in secrets.yaml
##########################################################################################
not_home_picture: # in entities card
  style:
    hui-generic-entity-row:
      $: |
        state-badge {
          filter: {{'grayscale(100%)' if not is_state(config.entity,'home')
                    else 'none'}};
        }

not_home_picture_glance:
  style: |
    state-badge {
      filter: {{'grayscale(100%)' if not is_state(config.entity,'home')
                else 'none'}};
        }

wind_direction:
  style: |
    :host {
      --card-mod-icon:
        {% set dir = states(config.entity)|int(default=0) %}
        {% set icons = ['down','bottom-left','left','top-left','up','top-right',
                        'right','bottom-right'] %}
        {% set quadrant = (state/45)|round %}
        {% if quadrant < icons|count %} mdi:arrow-{{icons[quadrant]}}
        {% else %} mdi:arrow-down
        {% endif %};
    }

wind_kracht:
  style: |
    :host {
        --paper-item-icon-color:
          {% set state = states(config.entity)|int(-5) %}
          {% set bft = ['lightblue','paleturquoise','aquamarine','greenyellow','lime',
                        'mediumspringgreen','yellowgreen','navy','gold','orange',
                        'tomato','orangered'] %}
          {{bft[state]}};
    }

wind_snelheid:
  style: |
    :host {
      --paper-item-icon-color:
        {% set state = (states(config.entity)|int(-5)) %}
        {% if state == 'unknown'%} gray
        {% elif state < 0.5 %} lightblue
        {% elif state < 1.5 %} paleturquoise
        {% elif state < 3.3 %} aquamarine
        {% elif state < 5.5 %} greenyellow
        {% elif state < 7.9 %} lime
        {% elif state < 10.7 %} mediumspringgreen
        {% elif state < 13.8 %} yellowgreen
        {% elif state < 17.1 %} navy
        {% elif state < 20.1 %} gold
        {% elif state < 24.4 %} orange
        {% elif state < 28.4 %} tomato
        {% elif state < 32.6 %} orangered
        {% else %} crimson
        {% endif %};
    }

temp_color:
  style: |
    :host {
      --paper-item-icon-color:
        {% set state = states(config.entity)|int(-5) %}
        {% if state == 'unknown'%} gray
        {% elif state < -20 %} black
        {% elif state < -15 %} navy
        {% elif state < -10 %} darkblue
        {% elif state < -5 %} mediumblue
        {% elif state < 0 %} blue
        {% elif state < 5 %} dodgerblue
        {% elif state < 10 %} lightblue
        {% elif state < 15 %} turquoise
        {% elif state < 20 %} green
        {% elif state < 25 %} darkgreen
        {% elif state < 30 %} orange
        {% elif state < 35 %} crimson
        {% else %} firebrick
        {% endif %};
    }

temp_binnen_color:
  style: |
    :host {
      --paper-item-icon-color:
        {% set state = states(config.entity)|int(-5) %}
        {% if state == 'unknown'%} gray
        {% elif state < 5 %} dodgerblue
        {% elif state < 10 %} lightblue
        {% elif state < 15 %} turquoise
        {% elif state < 20 %} green
        {% elif state < 25 %} darkgreen
        {% elif state < 30 %} orange
        {% elif state < 35 %} crimson
        {% else %} firebrick
        {% endif %};
    }

co2_color:
  style: |
    :host {
      --paper-item-icon-color:
        {% set state = states(config.entity)|int(-5) %}
        {% if state > 2100 %} purple
        {% elif state > 1600 %} red
        {% elif state > 1100 %} orange
        {% elif state > 900 %} brown
        {% else %} green
        {% endif %};
    }

wifi_signal_strength:
  style: |
    :host {
      --card-mod-icon:
        {% set signal = states(config.entity)|int(default=0) %}
        {% if signal == 'unknown' %} mdi:help
        {% elif signal >= -50 %} mdi:wifi-strength-4
        {% elif signal >= -60 %} mdi:wifi-strength-3
        {% elif signal >= -67 %} mdi:wifi-strength-2
        {% elif signal >= -70 %} mdi:wifi-strength-1
        {% elif signal >= -80 %} mdi:wifi-strength-outline
        {% else %} mdi:wifi-strength-alert-outline
        {% endif %};
      --paper-item-icon-color:
        {% if signal == 'unknown' %} gray
        {% elif signal >= -50 %} darkgreen
        {% elif signal >= -60 %} green
        {% elif signal >= -67 %} lightgreen
        {% elif signal >= -70 %} gold
        {% elif signal >= -80 %} orange
        {% else %} maroon
        {% endif %};
    }

illuminance_color:
  style: |
    :host {
      --paper-item-icon-color:
        {% set state = states(config.entity)|int(-5) %}
        {% if state < 1 %} black
        {% elif state < 2 %} firebrick
        {% elif state < 10 %} orange
        {% elif state < 50 %} green
        {% elif state < 150 %} gold
        {% elif state < 350 %} teal
        {% elif state < 700 %} dodgerblue
        {% elif state < 2000 %} lighskyblue
        {% elif state < 10000 %} lightblue
        {% else %} lightcyan
        {% endif %};
    }

zigbee_connectivity:
  style: |
    :host {
      --card-mod-icon:
        {% set con = states(config.entity) %}
        {{'mdi:zigbee' if con == 'connected' else 'mdi:wifi-strength-alert-outline'}};
      --paper-item-icon-color:
        {{'green' if con == 'connected' else 'maroon'}};
    }

style_volume:
  style: |
    :host {
      --card-mod-icon:
        {% set vol = states(config.entity)|float(0) %}
        {% if vol == 0.0 %} mdi:volume-off
        {% elif vol <= 0.3 %} mdi:volume-low
        {% elif vol <= 0.6  %} mdi:volume-medium
        {% else %} mdi:volume-high
        {% endif %};
      --paper-item-icon-color:
        {% if vol == 0.0 %} gray
        {% elif vol <= 0.3 %} dodgerblue
        {% elif vol <= 0.6 %} green
        {% elif vol <= 0.8 %} orange
        {% else %} red
        {% endif %};
    }

luchtvochtigheid_binnen:
  style: |
    :host {
      --card-mod-icon:
        {% set hum = states(config.entity)|float(0) %}
        mdi:water-percent{{'-alert' if hum > 60 or hum < 50}};
      --paper-item-icon-color:
        {% if hum > 80 %} red
        {% elif hum > 70 %} maroon
        {% elif hum > 60 %} orange
        {% elif hum > 49 %} green
        {% elif hum > 40 %} maroon
        {% else %} red
        {% endif %};
    }

power_color:
  style: |
    :host {
      --paper-item-icon-color:
        {% set state = states(config.entity)|float(-5) %}
        {% if state > 2000 %} purple
        {% elif state > 1000 %} maroon
        {% elif state > 450 %} darkred
        {% elif state > 300 %} firebrick
        {% elif state > 250 %} crimson
        {% elif state > 150 %} darkorange
        {% elif state > 70 %} orange
        {% elif state > 10 %} lightsalmon
        {% elif state > 0 %} gold
        {% else %} var(--no-power-color)
        {% endif %};
    }

player_icon:
  style: |
    :host {
      --card-mod-icon:
        {% set state = states(config.entity) %}
        {% set player = {
          'Hub':'mdi:tablet-dashboard',
          'Pod':'cil:apple-homepod-mini',
          'Woonkamer':'mdi:sofa',
          'Hall':'mdi:transit-transfer',
          'Master bedroom':'mdi:human-male-female',
          'Library':'mdi:microsoft-xbox',
          'Gym':'mdi:dumbbell',
          'Office Wijke':'cil:desktop-mac',
          'Marte':'mdi:human-child',
          'Studenten':'mdi:human-child',
          'TheBoss app':'mdi:home-assistant',
          'Symfonisk':'mdi:cast-audio'} %}
        {{player.get(state,'mdi:radio-tower')}};
    }
```
