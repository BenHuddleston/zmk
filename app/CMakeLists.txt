cmake_minimum_required(VERSION 3.13.1)

set(CONFIG_APPLICATION_DEFINED_SYSCALL true)
list(APPEND BOARD_ROOT ${CMAKE_SOURCE_DIR})
list(APPEND DTS_ROOT ${CMAKE_SOURCE_DIR})

# Add our custom Zephyr module for drivers w/ syscalls, etc.
list(APPEND DTS_ROOT ${CMAKE_SOURCE_DIR}/drivers/zephyr)

list(APPEND ZEPHYR_EXTRA_MODULES
  ${CMAKE_CURRENT_SOURCE_DIR}/drivers
)
list(APPEND SYSCALL_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/drivers/zephyr)

include(cmake/zmk_config.cmake)

# Find Zephyr. This also loads Zephyr's build system.
find_package(Zephyr REQUIRED HINTS ../zephyr)
project(zmk)

zephyr_linker_sources(RODATA include/linker/zmk-events.ld)

# Add your source file to the "app" target. This must come after
# find_package(Zephyr) which defines the target.
target_include_directories(app PRIVATE include)
target_sources_ifdef(CONFIG_ZMK_SLEEP app PRIVATE src/power.c)
target_sources(app PRIVATE src/kscan.c)
target_sources(app PRIVATE src/matrix_transform.c)
target_sources(app PRIVATE src/hid.c)
target_sources(app PRIVATE src/sensors.c)
target_sources_ifdef(CONFIG_ZMK_DISPLAY app PRIVATE src/display.c)
target_sources(app PRIVATE src/event_manager.c)
target_sources_ifdef(CONFIG_ZMK_BLE app PRIVATE src/ble_unpair_combo.c)
target_sources(app PRIVATE src/events/position_state_changed.c)
target_sources(app PRIVATE src/events/keycode_state_changed.c)
target_sources(app PRIVATE src/events/modifiers_state_changed.c)
target_sources(app PRIVATE src/events/sensor_event.c)
target_sources_ifdef(CONFIG_ZMK_BLE app PRIVATE src/events/ble_active_profile_changed.c)
if (NOT CONFIG_ZMK_SPLIT_BLE_ROLE_PERIPHERAL)
  target_sources(app PRIVATE src/behaviors/behavior_key_press.c)
  target_sources(app PRIVATE src/behaviors/behavior_reset.c)
  target_sources(app PRIVATE src/behaviors/behavior_hold_tap.c)
  target_sources(app PRIVATE src/behaviors/behavior_momentary_layer.c)
  target_sources(app PRIVATE src/behaviors/behavior_toggle_layer.c)
  target_sources(app PRIVATE src/behaviors/behavior_transparent.c)
  target_sources(app PRIVATE src/behaviors/behavior_none.c)
  target_sources(app PRIVATE src/behaviors/behavior_sensor_rotate_key_press.c)
  target_sources(app PRIVATE src/keymap.c)
endif()
target_sources_ifdef(CONFIG_ZMK_RGB_UNDERGLOW app PRIVATE src/behaviors/behavior_rgb_underglow.c)
target_sources_ifdef(CONFIG_ZMK_BLE app PRIVATE src/behaviors/behavior_bt.c)
target_sources_ifdef(CONFIG_ZMK_BLE app PRIVATE src/ble.c)
target_sources_ifdef(CONFIG_ZMK_SPLIT_BLE_ROLE_PERIPHERAL app PRIVATE src/split_listener.c)
target_sources_ifdef(CONFIG_ZMK_SPLIT_BLE_ROLE_PERIPHERAL app PRIVATE src/split/bluetooth/service.c)
target_sources_ifdef(CONFIG_ZMK_SPLIT_BLE_ROLE_CENTRAL app PRIVATE src/split/bluetooth/central.c)
target_sources_ifdef(CONFIG_ZMK_KSCAN_MOCK_DRIVER app PRIVATE src/kscan_mock.c)
target_sources_ifdef(CONFIG_ZMK_KSCAN_COMPOSITE_DRIVER app PRIVATE src/kscan_composite.c)
target_sources_ifdef(CONFIG_USB app PRIVATE src/usb.c)
target_sources_ifdef(CONFIG_ZMK_BLE app PRIVATE src/hog.c)
target_sources_ifdef(CONFIG_ZMK_RGB_UNDERGLOW app PRIVATE src/rgb_underglow.c)
target_sources(app PRIVATE src/endpoints.c)
target_sources(app PRIVATE src/hid_listener.c)
target_sources(app PRIVATE src/main.c)

zephyr_cc_option(-Wfatal-errors)
