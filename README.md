# blender-autorender-glbfolder

* Create a blender file in a folder of glbs
* add a camera, and light the scene

* Set the render settings to a resolution of 1024x1024.
* Get the directory of the current .blend file.
* Create a new collection named "Imported Objects" to hold the imported objects.
* Get the directory of the glb models.
* Get a list of all the glb files in the directory.
* Loop through each glb file.
* Import the glb file using the glTF importer.
* Set the active camera to the one in the scene.
* Move all selected objects to the "Imported Objects" collection.
* Set the render file path to the current directory with a file name based on the current index.
* Render the current scene to file.
* Select all objects in the "Imported Objects" collection and delete them.
* Update the progress in the console and progress bar.
* Remove the "Imported Objects" collection.
* Reset the progress bar.
* Print "Rendering complete!".

In summary, the script imports each glb file in the directory, moves the imported objects to a new collection, renders the scene to file, deletes the imported objects, and repeats the process for each glb file in the directory.

```python
import bpy
import os
import mathutils

# Set the render settings
bpy.context.scene.render.resolution_x = 1024
bpy.context.scene.render.resolution_y = 1024

# Get the directory of the current .blend file
blend_file_directory = bpy.path.abspath('//')

# Create a new collection to hold imported objects
collection = bpy.data.collections.new('Imported Objects')
bpy.context.scene.collection.children.link(collection)

# Get the directory of the glb models
glb_directory = os.path.join(blend_file_directory, 'glb_models')

# Get a list of all the glb files in the directory
glb_files = [f for f in os.listdir(glb_directory) if f.endswith('.glb')]

# Loop through each glb file
for index, glb_file in enumerate(glb_files):
    
    # Import the glb file
    bpy.ops.import_scene.gltf(filepath=os.path.join(glb_directory, glb_file))
    
    # Set the active camera
    cam_obj = bpy.context.scene.camera
    
    # Move imported objects to the new collection
    for obj in bpy.context.selected_objects:
        obj_collection = obj.users_collection[0]
        obj_collection.objects.unlink(obj)
        collection.objects.link(obj)
    
    # Take the shot
    bpy.context.scene.render.filepath = os.path.join(blend_file_directory, f"render_{index}.png")
    bpy.ops.render.render(write_still=True)
    
    # Remove the imported object
    for obj in collection.objects:
        obj.select_set(True)
    bpy.ops.object.delete(use_global=False)
    bpy.ops.outliner.orphans_purge()
    
    # Update progress in console
    print(f'Rendered {index+1}/{len(glb_files)}: {glb_file}')
    
    # Update progress in progress bar
    bpy.context.window_manager.progress_update(index/len(glb_files))
    
# Remove the collection
bpy.data.collections.remove(collection)

# Reset progress bar
bpy.context.window_manager.progress_update(0)

print("Rendering complete!")

```
