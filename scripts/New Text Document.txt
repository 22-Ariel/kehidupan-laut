import bpy
import math
import os

print("--- Memulai Setup Otomatisasi Simulasi Menstra ---")

# 1. AMBIL SEMUA OBJEK UTAMA
hiu = bpy.data.objects.get("Node")
kuda_laut = bpy.data.objects.get("Symmetry.1")
rumput1 = bpy.data.objects.get("Node.001")
rumput2 = bpy.data.objects.get("Node.002")
karang = bpy.data.objects.get("group1533979519.001")
gelembung_list = ["Node.004", "Node.005", "Node.006", "Node.007"]

semua_objek = [hiu, kuda_laut, rumput1, rumput2, karang]
for name in gelembung_list:
    g = bpy.data.objects.get(name)
    if g: semua_objek.append(g)

# Bersihkan animasi lama
for obj in semua_objek:
    if obj:
        obj.animation_data_clear()
        obj.rotation_mode = 'XYZ'

# Ambil posisi awal konstan
y_awal_hiu = hiu.location.y if hiu else 0.0
z_awal_kuda = kuda_laut.location.z if kuda_laut else 0.0
y_awal_kuda = kuda_laut.location.y if kuda_laut else 0.0

# 2. GENERATE ANIMASI PER FRAME (1-200)
for frame in range(1, 201):
    bpy.context.scene.frame_set(frame)
    
    # Animasi Hiu (Node) - Meliuk & Maju Mundur
    if hiu:
        hiu.location.y = y_awal_hiu + (1.5 - 1.5 * math.cos(2 * math.pi * frame / 200))
        hiu.rotation_euler.z = math.sin(frame * 0.25) * 0.15
        hiu.keyframe_insert(data_path="location", index=1)
        hiu.keyframe_insert(data_path="rotation_euler", index=2)
        
    # Animasi Kuda Laut (Symmetry.1) - Mengambang
    if kuda_laut:
        kuda_laut.location.z = z_awal_kuda + (math.sin(frame * 0.1) * 0.4)
        kuda_laut.location.y = y_awal_kuda + (math.cos(frame * 0.1) * 0.1)
        kuda_laut.keyframe_insert(data_path="location", index=2)
        kuda_laut.keyframe_insert(data_path="location", index=1)
        
    # Animasi Rumput Laut 1 & 2 - Bergoyang
    if rumput1:
        rumput1.rotation_euler.y = math.sin(frame * 0.08) * 0.15
        rumput1.rotation_euler.x = math.cos(frame * 0.08) * 0.05
        rumput1.keyframe_insert(data_path="rotation_euler", index=0)
        rumput1.keyframe_insert(data_path="rotation_euler", index=1)
    if rumput2:
        rumput2.rotation_euler.y = math.sin(frame * 0.07 + 2) * 0.12
        rumput2.rotation_euler.x = math.cos(frame * 0.07 + 2) * 0.04
        rumput2.keyframe_insert(data_path="rotation_euler", index=0)
        rumput2.keyframe_insert(data_path="rotation_euler", index=1)
        
    # Animasi Terumbu Karang - Goyang Lembut
    if karang:
        karang.rotation_euler.y = math.sin(frame * 0.04) * 0.05
        karang.rotation_euler.x = math.cos(frame * 0.04) * 0.02
        karang.keyframe_insert(data_path="rotation_euler", index=0)
        karang.keyframe_insert(data_path="rotation_euler", index=1)
        
    # Animasi Gelembung - Meluncur Naik
    for nama in gelembung_list:
        g = bpy.data.objects.get(nama)
        if g:
            if frame == 1:
                g.z_base = g.location.z
                g.x_base = g.location.x
                g.y_base = g.location.y
            g.location.z = g.z_base + ((frame / 200.0) * 8.0)
            ritme = frame * 0.15 + (int(nama[-1]) * 10)
            g.location.x = g.x_base + (math.sin(ritme) * 0.15)
            g.location.y = g.y_base + (math.cos(ritme) * 0.15)
            g.keyframe_insert(data_path="location")

# 3. LOAD AUDIO AUTOMATION
path_proyek = bpy.path.abspath("//")
path_suara = os.path.join(path_proyek, "sounds", "underwater_ambient.mp3")

if os.path.exists(path_suara):
    if not bpy.context.scene.sequence_editor:
        bpy.context.scene.sequence_editor_create()
    sequencer = bpy.context.scene.sequence_editor
    for strip in sequencer.sequences:
        if strip.type == 'SOUND': sequencer.sequences.remove(strip)
    suara_strip = sequencer.sequences.new_sound(name="SuaraLaut", filepath=path_suara, channel=1, frame_start=1)
    suara_strip.sound.use_memory_cache = True
    print("-> Audio berhasil dikoneksikan!")

print("--- Semua Animasi Bawah Air Siap dijalankan! ---")