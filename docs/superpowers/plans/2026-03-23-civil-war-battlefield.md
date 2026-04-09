# Civil War Battlefield — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a real-time tactical battle game in Godot 4 / GDScript inspired by North & South (1989) battlefield combat — Confederate vs Union with infantry, cavalry, and artillery.

**Architecture:** Scene-per-unit approach. Each unit type is its own scene (CharacterBody2D) with encapsulated behavior. The battlefield is a parent scene with TileMapLayer terrain. AI uses simple direction-based movement (NavigationAgent2D deferred to v2 since the battlefield is open with only one river obstacle — bridge provides the path). GameManager orchestrates game flow.

**Tech Stack:** Godot 4.x, GDScript, TileMapLayer, CharacterBody2D, Area2D (projectiles)

**Spec:** `docs/superpowers/specs/2026-03-23-civil-war-battlefield-design.md`

---

## File Structure

```
civil-war-battlefield/
├── project.godot
├── scenes/
│   ├── main.tscn                    # Root scene — assembles everything
│   ├── main.gd
│   ├── battlefield/
│   │   ├── battlefield.tscn         # Battlefield with terrain tilemap
│   │   └── battlefield.gd           # Terrain queries (is_hill, is_river, etc.)
│   ├── units/
│   │   ├── base_unit.gd             # Shared unit logic (health, damage, selection, detection, movement)
│   │   ├── infantry.tscn
│   │   ├── infantry.gd
│   │   ├── cavalry.tscn
│   │   ├── cavalry.gd
│   │   ├── artillery.tscn
│   │   ├── artillery.gd
│   │   ├── cannonball.tscn
│   │   └── cannonball.gd
│   ├── ui/
│   │   ├── hud.tscn
│   │   └── hud.gd
│   └── ai/
│       └── ai_controller.gd         # AI state machine for Confederate army
├── scripts/
│   └── game_manager.gd              # Game state, pause, unit selection
└── assets/
    └── icon.svg
```

---

## Task 0: Project Setup & Godot Installation

**Files:**
- Create: `civil-war-battlefield/project.godot`
- Create: `civil-war-battlefield/.gitignore`

- [ ] **Step 1: Install Godot 4**

Download Godot 4.x standard build (not .NET) from https://godotengine.org/download. On Windows, use WinGet or manual download:

```bash
# PowerShell:
winget install GodotEngine.GodotEngine
```

- [ ] **Step 2: Create project directory**

```bash
mkdir -p /home/freax/projects/github-repos/civil-war-battlefield/{scenes/{battlefield,units,ui,ai},scripts,assets}
```

- [ ] **Step 3: Create project.godot**

Create: `civil-war-battlefield/project.godot`

```ini
; Engine configuration file.
config_version=5

[application]

config/name="Civil War Battlefield"
run/main_scene="res://scenes/main.tscn"
config/features=PackedStringArray("4.4")

[display]

window/size/viewport_width=1280
window/size/viewport_height=720
window/stretch/mode="canvas_items"

[input]

move_up={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":87,"key_label":0,"unicode":119,"location":0,"echo":false,"script":null)
, Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":4194320,"key_label":0,"unicode":0,"location":0,"echo":false,"script":null)
]
}
move_down={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":83,"key_label":0,"unicode":115,"location":0,"echo":false,"script":null)
, Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":4194322,"key_label":0,"unicode":0,"location":0,"echo":false,"script":null)
]
}
move_left={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":65,"key_label":0,"unicode":97,"location":0,"echo":false,"script":null)
, Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":4194319,"key_label":0,"unicode":0,"location":0,"echo":false,"script":null)
]
}
move_right={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":68,"key_label":0,"unicode":100,"location":0,"echo":false,"script":null)
, Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":4194321,"key_label":0,"unicode":0,"location":0,"echo":false,"script":null)
]
}
select_unit_1={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":49,"key_label":0,"unicode":49,"location":0,"echo":false,"script":null)
]
}
select_unit_2={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":50,"key_label":0,"unicode":50,"location":0,"echo":false,"script":null)
]
}
select_unit_3={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":51,"key_label":0,"unicode":51,"location":0,"echo":false,"script":null)
]
}
cycle_unit={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":4194306,"key_label":0,"unicode":0,"location":0,"echo":false,"script":null)
]
}
pause={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":4194305,"key_label":0,"unicode":0,"location":0,"echo":false,"script":null)
]
}
restart={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":4194309,"key_label":0,"unicode":0,"location":0,"echo":false,"script":null)
, Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":32,"key_label":0,"unicode":32,"location":0,"echo":false,"script":null)
]
}

[rendering]

renderer/rendering_method="gl_compatibility"
```

- [ ] **Step 4: Initialize git repo**

```bash
cd /home/freax/projects/github-repos/civil-war-battlefield
echo ".godot/" > .gitignore
echo ".superpowers/" >> .gitignore
git init
git add -A
git commit -m "chore: initial Godot 4 project setup"
```

---

## Task 1: Battlefield Terrain

**Files:**
- Create: `scenes/battlefield/battlefield.tscn`
- Create: `scenes/battlefield/battlefield.gd`

- [ ] **Step 1: Create battlefield.gd**

```gdscript
extends Node2D

@onready var tilemap: TileMapLayer = $TileMapLayer

func _ready():
	add_to_group("battlefield")

func is_hill(world_pos: Vector2) -> bool:
	var cell = tilemap.local_to_map(tilemap.to_local(world_pos))
	var data = tilemap.get_cell_tile_data(cell)
	if data:
		return data.get_custom_data("terrain_type") == "hill"
	return false

func is_river(world_pos: Vector2) -> bool:
	var cell = tilemap.local_to_map(tilemap.to_local(world_pos))
	var data = tilemap.get_cell_tile_data(cell)
	if data:
		return data.get_custom_data("terrain_type") == "river"
	return false

func get_damage_multiplier(world_pos: Vector2) -> float:
	if is_hill(world_pos):
		return 0.75
	return 1.0
```

- [ ] **Step 2: Create battlefield.tscn**

```
[gd_scene load_steps=2 format=3]

[ext_resource type="Script" path="res://scenes/battlefield/battlefield.gd" id="1"]

[node name="Battlefield" type="Node2D"]
script = ExtResource("1")

[node name="TileMapLayer" type="TileMapLayer" parent="."]
```

**Note:** The tileset and tile painting must be done in the Godot editor:
1. Select TileMapLayer → create new TileSet (16x16 tile size)
2. Add TileSetAtlasSource — use a 4x1 colored image (64x16px): green=grass, brown=hill, blue=river, grey=bridge
3. Add custom data layer "terrain_type" (String) to TileSet
4. Set terrain_type per tile: "grass", "hill", "river", "bridge"
5. Add physics collision to river tiles only
6. Paint the map: grass base, vertical river through center, bridge crossing, hills upper-left and lower-right

- [ ] **Step 3: Commit**

```bash
git add scenes/battlefield/
git commit -m "feat: add battlefield scene with terrain tilemap and queries"
```

---

## Task 2: Base Unit Script

**Files:**
- Create: `scenes/units/base_unit.gd`

This is the shared superclass for all units. Handles health, damage, selection, detection, and common movement patterns. Subclasses only override what's different.

- [ ] **Step 1: Create base_unit.gd**

```gdscript
extends CharacterBody2D
class_name BaseUnit

signal died(unit: BaseUnit)
signal health_changed(unit: BaseUnit, new_health: float)

@export var unit_type: String = "base"
@export var max_health: float = 100.0
@export var speed: float = 120.0
@export var melee_damage: float = 10.0
@export var detection_radius: float = 80.0
@export var team: String = "union"
@export var strong_against: String = ""

var health: float
var is_selected: bool = false
var is_alive: bool = true
var current_target: BaseUnit = null
var is_player_controlled: bool = false

@onready var collision_shape: CollisionShape2D = $CollisionShape2D
@onready var detection_area: Area2D = $DetectionArea
@onready var sprite: Node2D = $Sprite  # ColorRect or Polygon2D

func _ready():
	health = max_health
	add_to_group(team)
	add_to_group("units")
	# Create unique detection shape per instance (avoid shared sub-resource mutation)
	var detect_shape = CircleShape2D.new()
	detect_shape.radius = detection_radius
	detection_area.get_node("CollisionShape2D").shape = detect_shape

func take_damage(amount: float, attacker: BaseUnit = null) -> void:
	if not is_alive:
		return
	# Hill defense
	var battlefield = get_tree().get_first_node_in_group("battlefield")
	if battlefield:
		amount *= battlefield.get_damage_multiplier(global_position)
	# Bonus damage from counter unit
	if attacker and attacker.strong_against == unit_type:
		amount *= 1.5
	health -= amount
	health_changed.emit(self, health)
	if health <= 0:
		health = 0
		die()

func die() -> void:
	is_alive = false
	died.emit(self)
	sprite.modulate = Color(0.3, 0.3, 0.3, 0.5)
	collision_shape.set_deferred("disabled", true)
	detection_area.monitoring = false
	detection_area.monitorable = false
	set_physics_process(false)

func set_selected(selected: bool) -> void:
	is_selected = selected
	modulate = Color(1.3, 1.3, 1.3) if selected else Color(1, 1, 1)

func get_enemy_group() -> String:
	return "confederate" if team == "union" else "union"

func find_nearest_enemy() -> BaseUnit:
	var enemies = get_tree().get_nodes_in_group(get_enemy_group())
	var nearest: BaseUnit = null
	var nearest_dist: float = INF
	for enemy in enemies:
		if enemy is BaseUnit and enemy.is_alive:
			var dist = global_position.distance_to(enemy.global_position)
			if dist < nearest_dist:
				nearest_dist = dist
				nearest = enemy
	return nearest

# --- Common movement (used by subclasses and AI) ---

func handle_player_input() -> void:
	var input_dir = Input.get_vector("move_left", "move_right", "move_up", "move_down")
	velocity = input_dir * speed

func move_toward_target(target: BaseUnit) -> void:
	var direction = (target.global_position - global_position).normalized()
	velocity = direction * speed

func apply_melee_damage(delta: float) -> void:
	for body in detection_area.get_overlapping_bodies():
		if body is BaseUnit and body.team != team and body.is_alive:
			if global_position.distance_to(body.global_position) < 40:
				body.take_damage(melee_damage * delta, self)
				if current_target == null:
					current_target = body

func _on_detection_area_body_entered(body: Node2D) -> void:
	if body is BaseUnit and body.team != team and body.is_alive:
		if current_target == null or not current_target.is_alive:
			current_target = body
```

- [ ] **Step 2: Commit**

```bash
git add scenes/units/base_unit.gd
git commit -m "feat: add base unit script with health, damage, selection, movement"
```

---

## Task 3: Infantry Unit

**Files:**
- Create: `scenes/units/infantry.tscn`
- Create: `scenes/units/infantry.gd`

- [ ] **Step 1: Create infantry.gd**

```gdscript
extends BaseUnit

func _ready():
	unit_type = "infantry"
	max_health = 100.0
	speed = 120.0
	melee_damage = 10.0
	detection_radius = 80.0
	strong_against = "artillery"
	super._ready()

func _physics_process(delta):
	if not is_alive:
		return

	if is_player_controlled and is_selected:
		handle_player_input()
	elif current_target and current_target.is_alive:
		move_toward_target(current_target)
	# else: velocity stays as set by AI controller (or zero)

	apply_melee_damage(delta)
	move_and_slide()
```

- [ ] **Step 2: Create infantry.tscn**

```
[gd_scene load_steps=4 format=3]

[ext_resource type="Script" path="res://scenes/units/infantry.gd" id="1"]

[sub_resource type="RectangleShape2D" id="body_shape"]
size = Vector2(32, 32)

[sub_resource type="CircleShape2D" id="detect_shape"]
radius = 80.0

[node name="Infantry" type="CharacterBody2D"]
collision_layer = 1
collision_mask = 1
script = ExtResource("1")

[node name="CollisionShape2D" type="CollisionShape2D" parent="."]
shape = SubResource("body_shape")

[node name="Sprite" type="ColorRect" parent="."]
offset_left = -16.0
offset_top = -16.0
offset_right = 16.0
offset_bottom = 16.0
color = Color(0.2, 0.4, 0.8, 1)

[node name="DetectionArea" type="Area2D" parent="."]
collision_layer = 0
collision_mask = 1
monitoring = true

[node name="CollisionShape2D" type="CollisionShape2D" parent="DetectionArea"]
shape = SubResource("detect_shape")

[node name="Label" type="Label" parent="."]
offset_left = -16.0
offset_top = -24.0
offset_right = 16.0
offset_bottom = -8.0
horizontal_alignment = 1
text = "INF"

[connection signal="body_entered" from="DetectionArea" to="." method="_on_detection_area_body_entered"]
```

- [ ] **Step 3: Commit**

```bash
git add scenes/units/infantry.*
git commit -m "feat: add infantry unit with melee combat"
```

---

## Task 4: Cavalry Unit

**Files:**
- Create: `scenes/units/cavalry.tscn`
- Create: `scenes/units/cavalry.gd`

- [ ] **Step 1: Create cavalry.gd**

```gdscript
extends BaseUnit

const CHARGE_DAMAGE: float = 20.0
const SUSTAINED_DAMAGE: float = 8.0
const CHARGE_COOLDOWN: float = 3.0

var charge_available: bool = true
var charge_timer: float = 0.0

func _ready():
	unit_type = "cavalry"
	max_health = 80.0
	speed = 200.0
	melee_damage = SUSTAINED_DAMAGE
	detection_radius = 80.0
	strong_against = "infantry"
	super._ready()

func _physics_process(delta):
	if not is_alive:
		return

	# Charge cooldown
	if not charge_available:
		charge_timer -= delta
		if charge_timer <= 0:
			charge_available = true

	if is_player_controlled and is_selected:
		handle_player_input()
	elif current_target and current_target.is_alive:
		move_toward_target(current_target)

	apply_charge_damage(delta)
	move_and_slide()

func apply_charge_damage(delta):
	for body in detection_area.get_overlapping_bodies():
		if body is BaseUnit and body.team != team and body.is_alive:
			if global_position.distance_to(body.global_position) < 40:
				if charge_available:
					body.take_damage(CHARGE_DAMAGE, self)
					charge_available = false
					charge_timer = CHARGE_COOLDOWN
				else:
					body.take_damage(SUSTAINED_DAMAGE * delta, self)
				if current_target == null:
					current_target = body
```

- [ ] **Step 2: Create cavalry.tscn**

Uses a Polygon2D triangle shape for visual differentiation per spec.

```
[gd_scene load_steps=4 format=3]

[ext_resource type="Script" path="res://scenes/units/cavalry.gd" id="1"]

[sub_resource type="RectangleShape2D" id="body_shape"]
size = Vector2(32, 32)

[sub_resource type="CircleShape2D" id="detect_shape"]
radius = 80.0

[node name="Cavalry" type="CharacterBody2D"]
collision_layer = 1
collision_mask = 1
script = ExtResource("1")

[node name="CollisionShape2D" type="CollisionShape2D" parent="."]
shape = SubResource("body_shape")

[node name="Sprite" type="Polygon2D" parent="."]
color = Color(0.2, 0.4, 0.8, 1)
polygon = PackedVector2Array(0, -18, 16, 14, -16, 14)

[node name="DetectionArea" type="Area2D" parent="."]
collision_layer = 0
collision_mask = 1
monitoring = true

[node name="CollisionShape2D" type="CollisionShape2D" parent="DetectionArea"]
shape = SubResource("detect_shape")

[node name="Label" type="Label" parent="."]
offset_left = -16.0
offset_top = -26.0
offset_right = 16.0
offset_bottom = -10.0
horizontal_alignment = 1
text = "CAV"

[connection signal="body_entered" from="DetectionArea" to="." method="_on_detection_area_body_entered"]
```

- [ ] **Step 3: Commit**

```bash
git add scenes/units/cavalry.*
git commit -m "feat: add cavalry unit with charge mechanic"
```

---

## Task 5: Artillery Unit & Cannonball

**Files:**
- Create: `scenes/units/artillery.tscn`
- Create: `scenes/units/artillery.gd`
- Create: `scenes/units/cannonball.tscn`
- Create: `scenes/units/cannonball.gd`

- [ ] **Step 1: Create cannonball.gd**

The cannonball carries a reference to the shooter so bonus damage (artillery strong_against cavalry) applies correctly.

```gdscript
extends Area2D
class_name Cannonball

var direction: Vector2 = Vector2.ZERO
var speed: float = 300.0
var damage: float = 25.0
var shooter: BaseUnit = null  # Reference to firing unit for bonus damage calc
var shooter_team: String = ""
var lifetime: float = 3.0

func _ready():
	body_entered.connect(_on_body_entered)

func _physics_process(delta):
	position += direction * speed * delta
	lifetime -= delta
	if lifetime <= 0:
		queue_free()

func _on_body_entered(body):
	if body is BaseUnit and body.team != shooter_team and body.is_alive:
		body.take_damage(damage, shooter)
		queue_free()
```

- [ ] **Step 2: Create cannonball.tscn**

```
[gd_scene load_steps=3 format=3]

[ext_resource type="Script" path="res://scenes/units/cannonball.gd" id="1"]

[sub_resource type="CircleShape2D" id="ball_shape"]
radius = 6.0

[node name="Cannonball" type="Area2D"]
collision_layer = 2
collision_mask = 1
script = ExtResource("1")

[node name="CollisionShape2D" type="CollisionShape2D" parent="."]
shape = SubResource("ball_shape")

[node name="Sprite" type="ColorRect" parent="."]
offset_left = -4.0
offset_top = -4.0
offset_right = 4.0
offset_bottom = 4.0
color = Color(0.1, 0.1, 0.1, 1)
```

- [ ] **Step 3: Create artillery.gd**

```gdscript
extends BaseUnit

const CANNONBALL_SCENE = preload("res://scenes/units/cannonball.tscn")
const FIRE_INTERVAL: float = 2.0
const RANGED_DETECTION: float = 400.0
const MELEE_FALLBACK_DAMAGE: float = 3.0
const CANNONBALL_DAMAGE: float = 25.0

var fire_timer: float = 0.0
var is_firing: bool = false

func _ready():
	unit_type = "artillery"
	max_health = 60.0
	speed = 40.0
	melee_damage = MELEE_FALLBACK_DAMAGE
	detection_radius = RANGED_DETECTION
	strong_against = "cavalry"
	super._ready()

func _physics_process(delta):
	if not is_alive:
		return

	fire_timer -= delta

	var ranged_target = find_ranged_target()

	if ranged_target:
		is_firing = true
		velocity = Vector2.ZERO  # Stationary when firing
		if fire_timer <= 0:
			fire_cannonball(ranged_target)
			fire_timer = FIRE_INTERVAL
	else:
		is_firing = false
		if is_player_controlled and is_selected:
			handle_player_input()
		elif current_target and current_target.is_alive:
			move_toward_target(current_target)

	# Melee fallback
	for body in detection_area.get_overlapping_bodies():
		if body is BaseUnit and body.team != team and body.is_alive:
			if global_position.distance_to(body.global_position) < 40:
				body.take_damage(MELEE_FALLBACK_DAMAGE * delta, self)

	move_and_slide()

func find_ranged_target() -> BaseUnit:
	var nearest = find_nearest_enemy()
	if nearest and global_position.distance_to(nearest.global_position) <= RANGED_DETECTION:
		return nearest
	return null

func fire_cannonball(target: BaseUnit) -> void:
	var ball = CANNONBALL_SCENE.instantiate()
	ball.global_position = global_position
	ball.direction = (target.global_position - global_position).normalized()
	ball.damage = CANNONBALL_DAMAGE
	ball.shooter = self  # Pass reference for bonus damage
	ball.shooter_team = team
	get_tree().current_scene.get_node("Projectiles").add_child(ball)
```

- [ ] **Step 4: Create artillery.tscn**

Uses a Polygon2D circle approximation for visual differentiation per spec.

```
[gd_scene load_steps=4 format=3]

[ext_resource type="Script" path="res://scenes/units/artillery.gd" id="1"]

[sub_resource type="RectangleShape2D" id="body_shape"]
size = Vector2(32, 32)

[sub_resource type="CircleShape2D" id="detect_shape"]
radius = 400.0

[node name="Artillery" type="CharacterBody2D"]
collision_layer = 1
collision_mask = 1
script = ExtResource("1")

[node name="CollisionShape2D" type="CollisionShape2D" parent="."]
shape = SubResource("body_shape")

[node name="Sprite" type="Polygon2D" parent="."]
color = Color(0.2, 0.4, 0.8, 1)
polygon = PackedVector2Array(16, 0, 11.3, 11.3, 0, 16, -11.3, 11.3, -16, 0, -11.3, -11.3, 0, -16, 11.3, -11.3)

[node name="DetectionArea" type="Area2D" parent="."]
collision_layer = 0
collision_mask = 1
monitoring = true

[node name="CollisionShape2D" type="CollisionShape2D" parent="DetectionArea"]
shape = SubResource("detect_shape")

[node name="Label" type="Label" parent="."]
offset_left = -16.0
offset_top = -24.0
offset_right = 16.0
offset_bottom = -8.0
horizontal_alignment = 1
text = "ART"

[connection signal="body_entered" from="DetectionArea" to="." method="_on_detection_area_body_entered"]
```

- [ ] **Step 5: Commit**

```bash
git add scenes/units/artillery.* scenes/units/cannonball.*
git commit -m "feat: add artillery with ranged cannonball attack and melee fallback"
```

---

## Task 6: Game Manager

**Files:**
- Create: `scripts/game_manager.gd`

- [ ] **Step 1: Create game_manager.gd**

```gdscript
extends Node

signal game_state_changed(new_state: String)
signal unit_selected(unit: BaseUnit)

enum GameState { SETUP, COUNTDOWN, BATTLE, PAUSED, VICTORY, DEFEAT, DRAW }

var state: GameState = GameState.SETUP
var selected_unit: BaseUnit = null
var player_units: Array[BaseUnit] = []
var enemy_units: Array[BaseUnit] = []
var countdown_timer: float = 2.0
var stalemate_timer: float = 15.0

func _ready():
	process_mode = Node.PROCESS_MODE_ALWAYS

func _process(delta):
	match state:
		GameState.COUNTDOWN:
			countdown_timer -= delta
			if countdown_timer <= 0:
				start_battle()
		GameState.BATTLE:
			check_victory_conditions()
			check_stalemate(delta)

func _unhandled_input(event):
	if event.is_action_pressed("pause") and (state == GameState.BATTLE or state == GameState.PAUSED):
		toggle_pause()
	elif event.is_action_pressed("restart") and state in [GameState.VICTORY, GameState.DEFEAT, GameState.DRAW]:
		restart_game()
	elif state == GameState.BATTLE:
		handle_unit_selection(event)

func setup_game(p_units: Array[BaseUnit], e_units: Array[BaseUnit]) -> void:
	player_units = p_units
	enemy_units = e_units

	for unit in player_units:
		unit.is_player_controlled = true
		unit.died.connect(_on_unit_died)
		unit.health_changed.connect(_on_damage_dealt)

	for unit in enemy_units:
		unit.is_player_controlled = false
		unit.died.connect(_on_unit_died)
		unit.health_changed.connect(_on_damage_dealt)

	if player_units.size() > 0:
		select_unit(player_units[0])

	state = GameState.COUNTDOWN
	countdown_timer = 2.0
	game_state_changed.emit("countdown")

func start_battle() -> void:
	state = GameState.BATTLE
	stalemate_timer = 15.0
	game_state_changed.emit("battle")

func handle_unit_selection(event: InputEvent) -> void:
	if event.is_action_pressed("select_unit_1"):
		select_unit_by_index(0)
	elif event.is_action_pressed("select_unit_2"):
		select_unit_by_index(1)
	elif event.is_action_pressed("select_unit_3"):
		select_unit_by_index(2)
	elif event.is_action_pressed("cycle_unit"):
		cycle_unit()

func select_unit_by_index(index: int) -> void:
	if index < player_units.size() and player_units[index].is_alive:
		select_unit(player_units[index])

func cycle_unit() -> void:
	var living = player_units.filter(func(u): return u.is_alive)
	if living.is_empty():
		return
	var current_index = living.find(selected_unit)
	var next_index = (current_index + 1) % living.size()
	select_unit(living[next_index])

func select_unit(unit: BaseUnit) -> void:
	if selected_unit:
		selected_unit.set_selected(false)
	selected_unit = unit
	selected_unit.set_selected(true)
	unit_selected.emit(unit)

func check_victory_conditions() -> void:
	var player_alive = player_units.filter(func(u): return u.is_alive)
	var enemy_alive = enemy_units.filter(func(u): return u.is_alive)
	if enemy_alive.is_empty():
		state = GameState.VICTORY
		game_state_changed.emit("victory")
	elif player_alive.is_empty():
		state = GameState.DEFEAT
		game_state_changed.emit("defeat")

func check_stalemate(delta: float) -> void:
	stalemate_timer -= delta
	if stalemate_timer <= 0:
		var p_hp = player_units.filter(func(u): return u.is_alive).reduce(func(acc, u): return acc + u.health, 0.0)
		var e_hp = enemy_units.filter(func(u): return u.is_alive).reduce(func(acc, u): return acc + u.health, 0.0)
		if p_hp > e_hp:
			state = GameState.VICTORY
			game_state_changed.emit("victory")
		elif e_hp > p_hp:
			state = GameState.DEFEAT
			game_state_changed.emit("defeat")
		else:
			state = GameState.DRAW
			game_state_changed.emit("draw")

func _on_damage_dealt(_unit: BaseUnit, _new_health: float) -> void:
	stalemate_timer = 15.0  # Reset on any damage

func _on_unit_died(_unit: BaseUnit) -> void:
	stalemate_timer = 15.0

func toggle_pause() -> void:
	if state == GameState.BATTLE:
		state = GameState.PAUSED
		get_tree().paused = true
		game_state_changed.emit("paused")
	elif state == GameState.PAUSED:
		state = GameState.BATTLE
		get_tree().paused = false
		game_state_changed.emit("battle")

func restart_game() -> void:
	get_tree().paused = false
	get_tree().reload_current_scene()
```

- [ ] **Step 2: Commit**

```bash
git add scripts/game_manager.gd
git commit -m "feat: add game manager with state machine, unit selection, stalemate detection"
```

---

## Task 7: HUD

**Files:**
- Create: `scenes/ui/hud.tscn`
- Create: `scenes/ui/hud.gd`

- [ ] **Step 1: Create hud.gd**

```gdscript
extends CanvasLayer

@onready var player_health_container: VBoxContainer = $PlayerHealth
@onready var enemy_health_container: VBoxContainer = $EnemyHealth
@onready var selected_label: Label = $SelectedUnit
@onready var center_message: Label = $CenterMessage

var health_bars: Dictionary = {}

func setup(p_units: Array[BaseUnit], e_units: Array[BaseUnit]) -> void:
	create_health_bars(p_units, player_health_container)
	create_health_bars(e_units, enemy_health_container)
	for unit in p_units + e_units:
		unit.health_changed.connect(_on_health_changed)

func create_health_bars(units: Array[BaseUnit], container: VBoxContainer) -> void:
	for unit in units:
		var hbox = HBoxContainer.new()
		var label = Label.new()
		label.text = unit.unit_type.substr(0, 3).to_upper()
		label.custom_minimum_size = Vector2(40, 0)
		hbox.add_child(label)
		var bar = ProgressBar.new()
		bar.max_value = unit.max_health
		bar.value = unit.health
		bar.custom_minimum_size = Vector2(100, 16)
		bar.show_percentage = false
		hbox.add_child(bar)
		container.add_child(hbox)
		health_bars[unit] = bar

func _on_health_changed(unit: BaseUnit, new_health: float) -> void:
	if unit in health_bars:
		health_bars[unit].value = new_health
		if new_health <= 0:
			health_bars[unit].modulate = Color(0.5, 0.5, 0.5, 0.5)

func set_selected_unit(unit: BaseUnit) -> void:
	selected_label.text = "Selected: " + unit.unit_type.to_upper()

func show_message(text: String, duration: float = 0.0) -> void:
	center_message.text = text
	center_message.visible = true
	if duration > 0:
		await get_tree().create_timer(duration).timeout
		center_message.visible = false

func hide_message() -> void:
	center_message.visible = false
```

- [ ] **Step 2: Create hud.tscn**

```
[gd_scene load_steps=2 format=3]

[ext_resource type="Script" path="res://scenes/ui/hud.gd" id="1"]

[node name="HUD" type="CanvasLayer"]
process_mode = 2
script = ExtResource("1")

[node name="PlayerHealth" type="VBoxContainer" parent="."]
offset_left = 10.0
offset_top = 10.0
offset_right = 200.0
offset_bottom = 100.0

[node name="PlayerLabel" type="Label" parent="PlayerHealth"]
text = "UNION"

[node name="EnemyHealth" type="VBoxContainer" parent="."]
anchors_preset = 1
anchor_left = 1.0
anchor_right = 1.0
offset_left = -200.0
offset_top = 10.0
offset_right = -10.0
offset_bottom = 100.0

[node name="EnemyLabel" type="Label" parent="EnemyHealth"]
text = "CONFEDERATE"
horizontal_alignment = 2

[node name="SelectedUnit" type="Label" parent="."]
anchors_preset = 7
anchor_left = 0.5
anchor_top = 1.0
anchor_right = 0.5
anchor_bottom = 1.0
offset_left = -80.0
offset_top = -40.0
offset_right = 80.0
offset_bottom = -10.0
horizontal_alignment = 1
text = "Selected: INFANTRY"

[node name="CenterMessage" type="Label" parent="."]
anchors_preset = 8
anchor_left = 0.5
anchor_top = 0.5
anchor_right = 0.5
anchor_bottom = 0.5
offset_left = -200.0
offset_top = -40.0
offset_right = 200.0
offset_bottom = 40.0
horizontal_alignment = 1
vertical_alignment = 1
theme_override_font_sizes/font_size = 32
text = ""
visible = false
```

- [ ] **Step 3: Commit**

```bash
git add scenes/ui/
git commit -m "feat: add HUD with health bars, unit selector, and messages"
```

---

## Task 8: AI Controller

**Files:**
- Create: `scenes/ai/ai_controller.gd`

The AI only sets `velocity` and `current_target` on units. It does NOT call `move_and_slide()` — each unit's own `_physics_process` handles that.

- [ ] **Step 1: Create ai_controller.gd**

```gdscript
extends Node

enum AIState { ADVANCE, ENGAGE, RETREAT, ARTILLERY_HOLD }

var units: Array[BaseUnit] = []
var unit_states: Dictionary = {}
var decision_timer: float = 0.0
const DECISION_INTERVAL: float = 0.5

func setup(ai_units: Array[BaseUnit]) -> void:
	units = ai_units
	for unit in units:
		unit_states[unit] = AIState.ADVANCE

func _physics_process(_delta):
	decision_timer -= _delta
	if decision_timer <= 0:
		decision_timer = DECISION_INTERVAL
		make_decisions()
	apply_velocities()

func make_decisions() -> void:
	for unit in units:
		if not unit.is_alive:
			continue
		unit_states[unit] = evaluate_unit(unit)

func evaluate_unit(unit: BaseUnit) -> AIState:
	if unit.unit_type == "artillery":
		return AIState.ARTILLERY_HOLD

	var living = units.filter(func(u): return u.is_alive)

	# Last unit: always engage
	if living.size() == 1:
		return AIState.ENGAGE

	# Low health: retreat (unless enemy is very close)
	if unit.health < unit.max_health * 0.3:
		var nearest = unit.find_nearest_enemy()
		if nearest and unit.global_position.distance_to(nearest.global_position) < 60:
			return AIState.ENGAGE
		return AIState.RETREAT

	# Enemy in range: engage
	var nearest = unit.find_nearest_enemy()
	if nearest and unit.global_position.distance_to(nearest.global_position) < unit.detection_radius * 2:
		return AIState.ENGAGE

	return AIState.ADVANCE

func apply_velocities() -> void:
	for unit in units:
		if not unit.is_alive:
			continue
		# Only set velocity for AI units that aren't targeting on their own
		var state = unit_states.get(unit, AIState.ADVANCE)
		match state:
			AIState.ADVANCE:
				var target_pos = Vector2(640, 360)
				var direction = (target_pos - unit.global_position).normalized()
				unit.velocity = direction * unit.speed * 0.6
			AIState.ENGAGE:
				var target = find_best_target(unit)
				if target:
					unit.current_target = target
				# Let the unit's own _physics_process handle movement toward current_target
			AIState.RETREAT:
				var retreat_pos = Vector2(100, unit.global_position.y)
				var direction = (retreat_pos - unit.global_position).normalized()
				unit.velocity = direction * unit.speed * 0.5
			AIState.ARTILLERY_HOLD:
				unit.velocity = Vector2.ZERO
				var nearest = unit.find_nearest_enemy()
				if nearest:
					unit.current_target = nearest

func find_best_target(unit: BaseUnit) -> BaseUnit:
	var enemies = get_tree().get_nodes_in_group(unit.get_enemy_group())
	var best: BaseUnit = null
	var best_score: float = -INF
	for enemy in enemies:
		if not enemy is BaseUnit or not enemy.is_alive:
			continue
		var score: float = 0.0
		var dist = unit.global_position.distance_to(enemy.global_position)
		if unit.strong_against == enemy.unit_type:
			score += 100.0
		score -= dist * 0.1
		score += (enemy.max_health - enemy.health) * 0.5
		if score > best_score:
			best_score = score
			best = enemy
	return best
```

- [ ] **Step 2: Commit**

```bash
git add scenes/ai/
git commit -m "feat: add AI controller with advance/engage/retreat/hold states"
```

---

## Task 9: Main Scene (Assembly)

**Files:**
- Create: `scenes/main.tscn`
- Create: `scenes/main.gd`

- [ ] **Step 1: Create main.gd**

Spawn positions follow the spec: vertical column, same X per side, different Y.
- Confederate (left): artillery at x=100 (rear), infantry at x=100, cavalry at x=100. Y offsets: cavalry=280 (front/closest to center), infantry=360, artillery=440 (rear).
- Union (right): cavalry at x=1180, y=280 (front), infantry at x=1180, y=360, artillery at x=1180, y=440 (rear).

```gdscript
extends Node2D

const INFANTRY_SCENE = preload("res://scenes/units/infantry.tscn")
const CAVALRY_SCENE = preload("res://scenes/units/cavalry.tscn")
const ARTILLERY_SCENE = preload("res://scenes/units/artillery.tscn")

@onready var game_manager: Node = $GameManager
@onready var hud: CanvasLayer = $HUD

const CONF_X = 100.0   # Confederate left side, 100px from edge
const UNION_X = 1180.0  # Union right side, 100px from edge
const CENTER_Y = 360.0
const SPACING = 80.0

func _ready():
	# Green background
	var bg = ColorRect.new()
	bg.color = Color(0.18, 0.35, 0.12)
	bg.size = Vector2(1280, 720)
	bg.z_index = -10
	add_child(bg)
	move_child(bg, 0)

	spawn_armies()

func spawn_armies() -> void:
	var player_units: Array[BaseUnit] = []
	var enemy_units: Array[BaseUnit] = []

	# Union (player) — right side, vertical column
	var u_infantry = spawn_unit(INFANTRY_SCENE, $UnionArmy, Vector2(UNION_X, CENTER_Y), "union")
	var u_cavalry = spawn_unit(CAVALRY_SCENE, $UnionArmy, Vector2(UNION_X, CENTER_Y - SPACING), "union")
	var u_artillery = spawn_unit(ARTILLERY_SCENE, $UnionArmy, Vector2(UNION_X, CENTER_Y + SPACING), "union")

	# Order matches 1/2/3 keys: infantry, cavalry, artillery
	player_units.append(u_infantry)
	player_units.append(u_cavalry)
	player_units.append(u_artillery)

	# Confederate (AI) — left side, vertical column
	var c_infantry = spawn_unit(INFANTRY_SCENE, $ConfederateArmy, Vector2(CONF_X, CENTER_Y), "confederate")
	var c_cavalry = spawn_unit(CAVALRY_SCENE, $ConfederateArmy, Vector2(CONF_X, CENTER_Y - SPACING), "confederate")
	var c_artillery = spawn_unit(ARTILLERY_SCENE, $ConfederateArmy, Vector2(CONF_X, CENTER_Y + SPACING), "confederate")

	enemy_units.append(c_infantry)
	enemy_units.append(c_cavalry)
	enemy_units.append(c_artillery)

	# Color confederate units red
	for unit in enemy_units:
		unit.get_node("Sprite").color = Color(0.8, 0.2, 0.2, 1)

	# Setup game manager
	game_manager.setup_game(player_units, enemy_units)
	game_manager.unit_selected.connect(hud.set_selected_unit)
	game_manager.game_state_changed.connect(_on_game_state_changed)

	# Setup HUD
	hud.setup(player_units, enemy_units)

	# Setup AI
	$AIController.setup(enemy_units)

func spawn_unit(scene: PackedScene, parent: Node2D, pos: Vector2, team_name: String) -> BaseUnit:
	var unit = scene.instantiate()
	unit.team = team_name
	unit.position = pos
	parent.add_child(unit)
	return unit

func _on_game_state_changed(new_state: String) -> void:
	match new_state:
		"countdown":
			hud.show_message("Battle Begins!", 2.0)
		"victory":
			hud.show_message("VICTORY!")
		"defeat":
			hud.show_message("DEFEAT!")
		"draw":
			hud.show_message("DRAW!")
		"paused":
			hud.show_message("PAUSED")
		"battle":
			hud.hide_message()
```

- [ ] **Step 2: Create main.tscn**

```
[gd_scene load_steps=5 format=3]

[ext_resource type="Script" path="res://scenes/main.gd" id="1"]
[ext_resource type="Script" path="res://scripts/game_manager.gd" id="2"]
[ext_resource type="PackedScene" path="res://scenes/battlefield/battlefield.tscn" id="3"]
[ext_resource type="PackedScene" path="res://scenes/ui/hud.tscn" id="4"]

[node name="Main" type="Node2D"]
script = ExtResource("1")

[node name="Battlefield" parent="." instance=ExtResource("3")]

[node name="UnionArmy" type="Node2D" parent="."]

[node name="ConfederateArmy" type="Node2D" parent="."]

[node name="Projectiles" type="Node2D" parent="."]

[node name="GameManager" type="Node" parent="."]
script = ExtResource("2")

[node name="AIController" type="Node" parent="."]

[node name="HUD" parent="." instance=ExtResource("4")]
```

Note: `AIController` script is set in main.gd or needs to be added:

Update main.tscn to include AI script:
```
[ext_resource type="Script" path="res://scenes/ai/ai_controller.gd" id="5"]

[node name="AIController" type="Node" parent="."]
script = ExtResource("5")
```

Final main.tscn with all 6 resources:

```
[gd_scene load_steps=6 format=3]

[ext_resource type="Script" path="res://scenes/main.gd" id="1"]
[ext_resource type="Script" path="res://scripts/game_manager.gd" id="2"]
[ext_resource type="PackedScene" path="res://scenes/battlefield/battlefield.tscn" id="3"]
[ext_resource type="PackedScene" path="res://scenes/ui/hud.tscn" id="4"]
[ext_resource type="Script" path="res://scenes/ai/ai_controller.gd" id="5"]

[node name="Main" type="Node2D"]
script = ExtResource("1")

[node name="Battlefield" parent="." instance=ExtResource("3")]

[node name="UnionArmy" type="Node2D" parent="."]

[node name="ConfederateArmy" type="Node2D" parent="."]

[node name="Projectiles" type="Node2D" parent="."]

[node name="GameManager" type="Node" parent="."]
script = ExtResource("2")

[node name="AIController" type="Node" parent="."]
script = ExtResource("5")

[node name="HUD" parent="." instance=ExtResource("4")]
```

- [ ] **Step 3: Commit**

```bash
git add scenes/main.tscn scenes/main.gd
git commit -m "feat: assemble main scene with armies, AI, HUD, and game manager"
```

---

## Task 10: Testing & Polish

- [ ] **Step 1: Open project in Godot editor**

1. Paint the tilemap (see Task 1 notes)
2. Run the project (F5)

- [ ] **Step 2: Verify core gameplay**

Test checklist:
- [ ] Units spawn: Confederate (red) left, Union (blue) right, vertical columns
- [ ] WASD/arrows move the selected unit
- [ ] 1/2/3 select infantry/cavalry/artillery; Tab cycles
- [ ] Dead units are skipped in selection
- [ ] AI units advance toward center
- [ ] Melee combat works on contact (health bars decrease)
- [ ] Artillery fires cannonballs at range
- [ ] Cavalry charge deals burst damage
- [ ] Bonus damage applies (cavalry > infantry, infantry > artillery, artillery > cavalry)
- [ ] Units die at 0 HP (greyed out, no collision)
- [ ] Victory/Defeat appears when one army is eliminated
- [ ] Stalemate triggers after 15s of no damage
- [ ] Escape pauses/unpauses
- [ ] Enter/Space restarts after game end
- [ ] Hill defense bonus reduces damage

- [ ] **Step 3: Fix any issues found during testing**

- [ ] **Step 4: Commit**

```bash
git add -A
git commit -m "feat: tilemap painted, playtested and polished"
```

---

## Task 11: GitHub Repo

- [ ] **Step 1: Create GitHub repo and push**

```bash
cd /home/freax/projects/github-repos/civil-war-battlefield
gh repo create freaxnx01/civil-war-battlefield --public --source=. --push
```

---

## Known Limitations (v2 candidates)

- **No pathfinding:** AI uses direct movement; units may get stuck on river if they don't aim for the bridge. Consider adding NavigationAgent2D in v2.
- **No sound:** Audio deferred per spec.
- **Placeholder visuals:** Colored shapes only; pixel art sprites are a natural next step.
- **Single map:** One battlefield layout; v2 could add multiple maps.
