# Pangolin undeclared function/keywords

github.com/facebookresearch/DeepSDF/issues/81

# 删除`sudo make install`的安装包

```bash
  sudo rm -rf /usr/local/include/pangolin
  sudo rm -rf /usr/local/lib/libpangolin*
  sudo rm -rf /usr/local/lib/cmake/Pangolin
```

# 处理非"watertight"和"volume<0"
## trimesh
```python
import trimesh
import numpy as np

def repair_and_process(mesh_filepath, target_filepath, repaired_path=None):
    # 1. 加载
    mesh = trimesh.load(mesh_filepath, force='mesh')
    print(f"修复前 - 顶点: {len(mesh.vertices)}, 面片: {len(mesh.faces)}")
    print(f"修复前 - 是否水密: {mesh.is_watertight}, 体积: {mesh.volume}")

    # 2. 修复步骤
    # a. 移除重复面和非法值
    mesh.remove_duplicate_faces()
    mesh.remove_infinite_values()
    mesh.remove_degenerate_faces()
    
    # b. 修复法线方向（强制统一）
    trimesh.repair.fix_normals(mesh, multibody=True)
    
    # c. 尝试填充孔洞（对水密性很重要）
    trimesh.repair.fill_holes(mesh)
    
    # d. 如果仍然破碎，提取最大连通分量
    if not mesh.is_watertight:
        print("网格仍不水密，尝试提取最大连通分量...")
        mesh = mesh.split(only_watertight=False)[0]  # 取最大的一个子网格
    
    # e. 最后统一法线方向
    mesh.fix_normals()
    
    print(f"修复后 - 是否水密: {mesh.is_watertight}, 体积: {abs(mesh.volume)}")
    
    mesh.export(repaired_path)

    # 3. 采样（大幅减少采样数以测试）
    points, sdf = sample_sdf_near_surface(
        mesh,
        number_of_points=100000,  # 先用10万点测试
        surface_point_method='scan',
        scan_count=30,            # 减少扫描次数
        scan_resolution=300,      # 降低分辨率
        sign_method='depth'       # 尝试使用深度法而非法线法（对破碎网格更鲁棒）
    )
    
    # 4. 保存
    results = np.concatenate([points, np.expand_dims(sdf, axis=-1)], axis=1)
    np.savez_compressed(target_filepath, 
                        pos=results[sdf >= 0], 
                        neg=results[sdf < 0])
    print(f"保存到: {target_filepath}")
    return mesh  # 返回修复后的网格供检查
```
## open3d

```python
import open3d as o3d

def open3d_repair(mesh_filepath, repaired_filepath):
    """使用Open3D的强力修复算法"""
    mesh = o3d.io.read_triangle_mesh(mesh_filepath)
    
    vertices = np.asarray(mesh.vertices)
    faces = np.asarray(mesh.triangles)
    original_mesh = trimesh.Trimesh(vertices=vertices, faces=faces)
    if original_mesh.volume>=0  and original_mesh.is_watertight:
        print("mesh is ok: ", mesh_filepath)
        return  

    # 修复（Open3D的修复通常更鲁棒）
    mesh.remove_duplicated_vertices()
    mesh.remove_duplicated_triangles()
    mesh.remove_degenerate_triangles()
    mesh.remove_unreferenced_vertices()

    # 尝试计算顶点法线
    mesh.compute_vertex_normals()
    
    # 转换为trimesh格式以便后续处理
    vertices = np.asarray(mesh.vertices)
    faces = np.asarray(mesh.triangles)
    repaired_mesh = trimesh.Trimesh(vertices=vertices, faces=faces)
    
    print(f"修复后 - 是否水密: {repaired_mesh.is_watertight}, 体积: {abs(repaired_mesh.volume)}")
    repaired_mesh.export(repaired_filepath)

    return
```
