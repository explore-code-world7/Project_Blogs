- 神经网络设置好，就是可以正常跑的

# runtimeerror CUDA error: an illegal memory access was encountered

> runtimeerror CUDA error: an illegal memory access was encounteredCUDA kernel errors might be asynchronously reported at some other API call, so the stacktrace below might be incorrect.For debugging consider passing CUDA_LAUNCH_BLOCKING=1Compile with `TORCH_USE_CUDA_DSA` to enable device-side assertions.

## solution
1. check reset_root_states()& reset_dof_states()
* remember to create assets in order, be it actor or non-actor asset;

# Expected parameter loc of distribution Normal(loc: , scale: ) to satisfy the constraint Real(), but found invalid values: with plenty of nan appearing
1. 系数爆炸
2. 对坟墓加laplace项
3. 地形坐标没选好, 越界 
