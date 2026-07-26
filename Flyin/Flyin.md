# **What is it?**

Move drones from **point A** to **point B** through a network of zones, as fast as possible.

---

### **Key Concepts**

Graph:

- _A network of **zones** (nodes) connected by **paths** (edges)_
- _Like a map with locations and roads between them_

**🚁 Drones**

- _Multiple drones start at the same zone_
- _All must reach the end zone_
- _They can move simultaneously_

**📍 Zone Types**

|Type|Meaning|
|---|---|
|`normal`|costs 1 turn|
|`priority`|costs 1 turn, preferred|
|`restricted`|costs 2 turns|
|`blocked`|cannot enter|

### ⏱️ Turns

- Simulation runs **step by step**
- Each step = 1 turn
- Goal = finish in **minimum turns**

### 🚦 Capacity Rules

- Each zone has a **max drones limit**
- Each connection has a **max traffic limit**
- Drones **wait** if zone is full

---

## What You Build

1. **Parser** → read the map file
2. **Graph** → store zones and connections
3. **Pathfinder** → find best routes (BFS/A*)
4. **Simulation** → move drones turn by turn
5. **Visualizer** → show the simulation visually

Git hub : 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

![[Flyin.png]]

```python
trun = 0
        while True:
            zone_ocpy: dict[str, int] = {}
            move = []
            trun += 1
            for drone in self.drones:
                if drone.step + 1 < len(drone.path):
                    next_drone = drone.path[drone.step + 1]
                    current_drone = zone_ocpy.get(next_drone, 0)
                    max_cap = self.graph.zones[next_drone].max_drones
                    if current_drone < max_cap:
                        zone_ocpy[next_drone] = current_drone + 1
                        drone.step += 1
                        drone.position = next_drone
                        move.append(f"{drone.id}-{drone.position}")
            if move:
                # print(" ".join(move))
                self.visualizer.print_trun(trun, move, self.graph)

            if all(drone.step == len(drone.path) - 1 for drone in self.drones):
                break
```