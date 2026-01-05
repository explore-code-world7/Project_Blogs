
# pool

```python
from multiprocessing import Pool    # 新加速

with Pool(processes=concurrent_processes) as pool:
    # 使用 map 方法将函数应用到每个网格文件
    # 这里会阻塞，直到所有网格处理完毕
    # results = pool.starmap(process_mesh, 
    #     [ (mesh_filepath, target_filepath, executable, specific_args + additional_general_args) \
    #         for (mesh_filepath, target_filepath, specific_args) in meshes_targets_and_specific_args])
    
    # executed by py
    # results = pool.starmap(process_single_mesh, 
    #                     [ (mesh_filepath, target_filepath, False) for (mesh_filepath, target_filepath, specific_args) in meshes_targets_and_specific_args])

    for i, batch in enumerate(batched([ (mesh_filepath, target_filepath, False) \
        for (mesh_filepath, target_filepath, specific_args) in meshes_targets_and_specific_args], batch_size)):
        print(f"处理第 {i+1} 批，共 {len(batch)} 个任务...")
        results_batch = pool.starmap(process_single_mesh, batch)
        print(f"第 {i+1} 批完成")
```



# ProcessPoolExecutor

```python
import concurrent.futures
 
def add(a, b):
    return a + b
 
if __name__ == '__main__':
    # 准备参数
    args = [(1, 2), (3, 4), (5, 6)]
    with concurrent.futures.ProcessPoolExecutor() as executor:
        results = executor.map(lambda x: add(*x), args)
        for result in results:
            print(result)
```
