# Add RGB, RGBW and RGBWW capability to template.light
## What is this ?

This replace the default `template` components with on that fix a deprecation of `template.light`

`Template.light` could nolonger be used to controle RGBW light since #76923 and the deprecation of a "white control" for light.
The documentation also wasn't updated accordingly to this change
This bring full RGB, RGBW and RGBWW capability in addition the the already existing TEMPERATURE and HS capability.
It fix the previous regression and expend on it.

This can be used to template RGB(WW) light composed of multiple entity into a single one for exemple. (see `Make a Global light entity for a multi segments WLED light` below`)

From https://github.com/home-assistant/core/pull/86047 and https://github.com/home-assistant/home-assistant.io/pull/25994.
Since the MR to fix this are stale and haven't been merged, i made this HACS repo to share it with other.

## How to build
To build this components, first make sure that the source branch have been rebased on the version you wish to build for.
If breaking change happened and you want to build for an older version, you may need to edit the playbook to point it to a fork with a branch at the latest compatible commit.

Then make a python virtual env to install the requirement file, and get inside
```bash
python3 -m venv ./venv
source venv/bin/activate
```
Install the required pip pkgs.
```bash
pip install -r requirements.txt
```
Now, you can set the `home_assistant_source_version` variable to the home assistant release you wish to target.
You can do so in the playbook.yml, or any other way supported by ansible.

Build the components with `ansible-playbook playbook.yml`.
Test it, it is found in the `custom_components` folder.
Then push the change and the created tags.
```bash
git push
git push origin 202X.X.XvX
```
Don't forget to create a release from the github page.

## Documentation update

The `template` platform creates lights that combine integrations and provides the
ability to run scripts or invoke services for each of the on, off, and
brightness commands of a light.

To enable Template Lights in your installation, add the following to your
`configuration.yaml` file:

```yaml
# Example configuration.yaml entry
light:
  - platform: template
    lights:
      theater_lights:
        friendly_name: "Theater Lights"
        level_template: "{{ state_attr('sensor.theater_brightness', 'lux')|int }}"
        value_template: "{{ state_attr('sensor.theater_brightness', 'lux')|int > 0 }}"
        temperature_template: "{{states('input_number.temperature_input') | int}}"
        hs_template: "({{states('input_number.h_input') | int}}, {{states('input_number.s_input') | int}})"
        effect_list_template: "{{ state_attr('light.led_strip', 'effect_list') }}"
        turn_on:
          service: script.theater_lights_on
        turn_off:
          service: script.theater_lights_off
        set_level:
          service: script.theater_lights_level
          data:
            brightness: "{{ brightness }}"
        set_temperature:
          service: input_number.set_value
          data:
            value: "{{ color_temp }}"
            entity_id: input_number.temperature_input
        set_hs:
          - service: input_number.set_value
            data:
              value: "{{ h }}"
              entity_id: input_number.h_input
          - service: input_number.set_value
            data:
              value: "{{ s }}"
              entity_id: input_number.s_input
          - service: light.turn_on
            data_template:
              entity_id:
                - light.led_strip
              transition: "{{ transition | float }}"
              hs_color:
                - "{{ hs[0] }}"
                - "{{ hs[1] }}"
        set_effect:
          - service: light.turn_on
            data_template:
              entity_id:
                - light.led_strip
              effect: "{{ effect }}"
        supports_transition_template: "{{ true }}"
```

``` yaml
  lights:
    description: List of your lights.
    required: true
    type: map
    keys:
      friendly_name:
        description: Name to use in the frontend.
        required: false
        type: string
      unique_id:
        description: An ID that uniquely identifies this light. Set this to a unique value to allow customization through the UI.
        required: false
        type: string
      value_template:
        description: Defines a template to get the state of the light.
        required: false
        type: template
        default: optimistic
      level_template:
        description: Defines a template to get the brightness of the light.
        required: false
        type: template
        default: optimistic
      temperature_template:
        description: Defines a template to get the color temperature of the light.
        required: false
        type: template
        default: optimistic
      hs_template:
        description: Defines a template to get the hs color of the light. Must render a tuple (hue, saturation)
        required: false
        type: template
        default: optimistic
      rgb_template:
        description: Defines a template to get the rgb color of the light. Must render a tuple or a list (red, green, blue)
        required: false
        type: template
        default: optimistic
      rgbw_template:
        description: Defines a template to get the rgbw color of the light. Must render a tuple or a list (red, green, blue, white)
        required: false
        type: template
        default: optimistic
      rgbww_template:
        description: Defines a template to get the rgbww color of the light. Must render a tuple or a list (red, green, blue, cold white, warm white)
        required: false
        type: template
        default: optimistic
      supports_transition_template:
        description: Defines a template to get if light supports transition. Should return boolean value (True/False). If this value is `True` transition parameter in a turn on or turn off call will be passed as a named parameter `transition` to either of the scripts.
        required: false
        type: template
        default: false
      effect_list_template:
        description: Defines a template to get the list of supported effects. Must render a list
        required: inclusive
        type: template
        default: optimistic
      effect_template:
        description: Defines a template to get the effect of the light.
        required: inclusive
        type: template
        default: optimistic
      min_mireds_template:
        description: Defines a template to get the min mireds value of the light.
        required: false
        type: template
        default: optimistic
      max_mireds_template:
        description: Defines a template to get the max mireds value of the light.
        required: false
        type: template
        default: optimistic
      icon_template:
        description: Defines a template for an icon or picture, e.g.,  showing a different icon for different states.
        required: false
        type: template
      availability_template:
        description: Defines a template to get the `available` state of the entity. If the template either fails to render or returns `True`, `"1"`, `"true"`, `"yes"`, `"on"`, `"enable"`, or a non-zero number, the entity will be `available`. If the template returns any other value, the entity will be `unavailable`. If not configured, the entity will always be `available`. Note that the string comparison not case sensitive; `"TrUe"` and `"yEs"` are allowed.
        required: false
        type: template
        default: true
      turn_on:
        description: Defines an action to run when the light is turned on.
        required: true
        type: action
      turn_off:
        description: Defines an action to run when the light is turned off.
        required: true
        type: action
      set_level:
        description: Defines an action to run when the light is given a brightness command. The script will only be called if the `turn_on` call only has brightness, and optionally transition.
        required: false
        type: action
      set_temperature:
        description: Defines an action to run when the light is given a color temperature command.
        required: false
        type: action
      set_hs:
        description: "Defines an action to run when the light is given a hs color command. Available variables: `hs` as a tuple, `h` and `s`"
        required: false
        type: action
      set_rgb:
        description: "Defines an action to run when the light is given a rgb color command. Available variables: `rgb` as a tuple, `r`, `g` and `b`"
        required: false
        type: action
      set_rgbw:
        description: "Defines an action to run when the light is given a rgbw color command. Available variables: `rgbw` as a tuple, `rgb` as a tuple, `r`, `g`, `b` and `w`"
        required: false
        type: action
      set_rgbww:
        description: "Defines an action to run when the light is given a rgbww color command. Available variables: `rgbww` as a tuple, `rgb` as a tuple, `r`, `g`, `b`, `cw` and `ww`"
        required: false
        required: false
        type: action
      set_effect:
        description: Defines an action to run when the light is given an effect command.
        required: inclusive
        type: action
```

### Template and action variables

State-based template entities have the special template variable `this` available in their templates and actions. The `this` variable aids [self-referencing](/integrations/template#self-referencing) of an entity's state and attribute in templates and actions.

## Considerations

If you are using the state of a platform that takes extra time to load, the
Template Light may get an `unknown` state during startup. This results
in error messages in your log file until that platform has completed loading.
If you use `is_state()` function in your template, you can avoid this situation.
For example, you would replace
{% raw %}`{{ states.switch.source.state == 'on' }}`{% endraw %}
with this equivalent that returns `true`/`false` and never gives an unknown
result:
{% raw %}`{{ is_state('switch.source', 'on') }}`{% endraw %}
Transition doesn't have its own script, it will instead be passed as a named parameter `transition` to the `turn_on`, `turn_off`, `brightness`, `color_temp`, `effect`, `hs_color`, `rgb_color`, `rgbw_color` or `rgbww_color` scripts.
Brightness will be passed as a named parameter `brightness` to either of `turn_on`, `color_temp`, `effect`, `hs_color`, `rgb_color`, `rgbw_color` or `rgbww_color` scripts if the corresponding parameter is also in the call. In this case the brightness script (`set_level`) will not be called.
If only brightness is passed to `light.turn_on` service call then `set_level` script is called.

## Examples

In this section you will find some real-life examples of how to use this light.

### Theater Volume Control

This example shows a light that is actually a home theater's volume. This
component gives you the flexibility to provide whatever you'd like to send as
the payload to the consumer including any scale conversions you may need to
make; the [Media Player component](/integrations/media_player/) needs a floating
point percentage value from `0.0` to `1.0`.

```yaml
light:
  - platform: template
    lights:
      theater_volume:
        friendly_name: "Receiver Volume"
        value_template: >-
          {% if is_state('media_player.receiver', 'on') %}
            {% if state_attr('media_player.receiver', 'is_volume_muted') %}
              off
            {% else %}
              on
            {% endif %}
          {% else %}
            off
          {% endif %}
        turn_on:
          service: media_player.volume_mute
          target:
            entity_id: media_player.receiver
          data:
            is_volume_muted: false
        turn_off:
          service: media_player.volume_mute
          target:
            entity_id: media_player.receiver
          data:
            is_volume_muted: true
        set_level:
          service: media_player.volume_set
          target:
            entity_id: media_player.receiver
          data:
            volume_level: "{{ (brightness / 255 * 100)|int / 100 }}"
        level_template: >-
          {% if is_state('media_player.receiver', 'on') %}
            {{ (state_attr('media_player.receiver', 'volume_level')|float * 255)|int }}
          {% else %}
            0
          {% endif %}
```


### Change The Icon

This example shows how to change the icon based on the light state.


```yaml
light:
  - platform: template
    lights:
      theater_volume:
        friendly_name: "Receiver Volume"
        value_template: >-
          {% if is_state('media_player.receiver', 'on') %}
            {% if state_attr('media_player.receiver', 'is_volume_muted') %}
              off
            {% else %}
              on
            {% endif %}
          {% else %}
            off
          {% endif %}
        icon_template: >-
          {% if is_state('media_player.receiver', 'on') %}
            {% if state_attr('media_player.receiver', 'is_volume_muted') %}
              mdi:lightbulb-off
            {% else %}
              mdi:lightbulb-on
            {% endif %}
          {% else %}
            mdi:lightbulb-off
          {% endif %}
        turn_on:
          service: media_player.volume_mute
          target:
            entity_id: media_player.receiver
          data:
            is_volume_muted: false
        turn_off:
          service: media_player.volume_mute
          target:
            entity_id: media_player.receiver
          data:
            is_volume_muted: true
```


### Change The Entity Picture

This example shows how to change the entity picture based on the light state.

```yaml
light:
  - platform: template
    lights:
      theater_volume:
        friendly_name: "Receiver Volume"
        value_template: >-
          {% if is_state('media_player.receiver', 'on') %}
            {% if state_attr('media_player.receiver', 'is_volume_muted') %}
              off
            {% else %}
              on
            {% endif %}
          {% else %}
            off
          {% endif %}
        icon_template: >-
          {% if is_state('media_player.receiver', 'on') %}
            {% if state_attr('media_player.receiver', 'is_volume_muted') %}
              /local/lightbulb-off.png
            {% else %}
              /local/lightbulb-on.png
            {% endif %}
          {% else %}
            /local/lightbulb-off.png
          {% endif %}
        turn_on:
          service: media_player.volume_mute
          target:
            entity_id: media_player.receiver
          data:
            is_volume_muted: false
        turn_off:
          service: media_player.volume_mute
          target:
            entity_id: media_player.receiver
          data:
            is_volume_muted: true
```

### Make a Global light entity for a multi segments WLED light

This example shows how to group together 2 RGBW segments from the same WLED controller into a single usable light

```yaml
light:
  - platform: template
    lights:
      wled_global:
        unique_id: 28208f257b54c44e50deb2d618d44710
        friendly_name: "Multi-segment Wled control"
        value_template: "{{ states('light.wled_master') }}"
        level_template: "{{ state_attr('light.wled_master', 'brightness')|d(0,true)|int }}"
        rgbw_template: (
          {{ (state_attr('light.wled_segment_0', 'rgbw_color')[0]|d(0) + state_attr('light.wled_segment_1', 'rgbw_color')[0]|d(0))/2 }},
          {{ (state_attr('light.wled_segment_0', 'rgbw_color')[1]|d(0) + state_attr('light.wled_segment_1', 'rgbw_color')[1]|d(0))/2 }},
          {{ (state_attr('light.wled_segment_0', 'rgbw_color')[2]|d(0) + state_attr('light.wled_segment_1', 'rgbw_color')[2]|d(0))/2 }},
          {{ (state_attr('light.wled_segment_0', 'rgbw_color')[3]|d(0) + state_attr('light.wled_segment_1', 'rgbw_color')[3]|d(0))/2 }}
          )
        effect_list_template: "{{ state_attr('light.wled_segment_0', 'effect_list') }}"
        effect_template: "{{ state_attr('light.wled_segment_0', 'effect') if state_attr('light.wled_segment_0', 'effect') == state_attr('light.wled_segment_1', 'effect') else none }}"
        availability_template: "{{ not is_state('light.wled_master', 'unknown') }}"

        turn_on:
          service: light.turn_on
          entity_id: light.wled_segment_0, light.wled_segment_1, light.wled_master
        turn_off:
          service: light.turn_off
          entity_id: light.wled_master
        set_level:
          service: light.turn_on
          entity_id: light.wled_master
          data_template:
            brightness: "{{ brightness }}"
        set_rgbw:
          service: light.turn_on
          entity_id: light.wled_segment_0, light.wled_segment_1
          data_template:
            rgbw_color:
              - "{{ r }}"
              - "{{ g }}"
              - "{{ b }}"
              - "{{ w }}"
            effect: "Solid"
        set_effect:
          service: light.turn_on
          entity_id: light.wled_segment_0, light.wled_segment_1
          data_template:
            effect: "{{ effect }}"
```
