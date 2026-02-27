# render
> output rendered result from input
* render_batch
```bash
    def render_batch(
        self,
        Rs,
        ts,
        models,
        *,
        Ks,
        width,
        height,
        znear=0.01,
        zfar=100,
        rot_type="mat",
        mode=["color", "depth"],
    ):
        """render a batch (vertex color), each contain one object
        Args:
            Rs (tensor): [b,3,3] or [b,4]
            ts (tensor): [b,3,]
            models (list of dicts): each stores {"vertices":, "colors":, "faces":, }
            Ks (tensor): [b,3,3]
            mode: color, depth, mask, xyz (one or more must be given)
        Returns:
            dict:
                color: bhw3
                mask: bhw
                depth: bhw
                xyz: bhw3
                probs: bhw
        """
        assert self.dib_ren.mode in ["VertexColorBatch"], self.dib_ren.mode
        ret = {}
        self.dib_ren.set_camera_parameters_from_RT_K(Rs, ts, Ks, height, width, near=znear, far=zfar, rot_type=rot_type)

        colors = [model["colors"][None] for model in models]  # b x [1, p, 3]
        points = [[model["vertices"][None], model["faces"].long()] for model in models]

        # points: list of [vertices, faces]
        # colors: list of colors
        color, im_prob, _, im_mask = self.dib_ren.forward(points=points, colors=colors)
        ret["color"] = color
        ret["prob"] = im_prob.squeeze(-1)
        ret["mask"] = im_mask.squeeze(-1)

        if "depth" in mode:
            # transform xyz
            if not isinstance(Rs, torch.Tensor):
                Rs = torch.stack(Rs)  # list
            if rot_type == "quat":
                R_mats = quat2mat_torch(Rs)
            else:
                R_mats = Rs
            xyzs = [
                misc.transform_pts_Rt_th(model["vertices"], R_mats[_id], ts[_id])[None]
                for _id, model in enumerate(models)
            ]
            ren_xyzs, _, _, _ = self.dib_ren.forward(points=points, colors=xyzs)
            ret["depth"] = ren_xyzs[:, :, :, 2]  # bhw

        if "xyz" in mode:  # TODO: check this
            obj_xyzs = [model["vertices"][None] for _id, model in enumerate(models)]
            ren_obj_xyzs, _, _, _ = self.dib_ren.forward(points=points, colors=obj_xyzs)
            ret["xyz"] = ren_obj_xyzs
        return ret
```
* render_batch_tex
```bash
    def render_batch_tex(
        self,
        Rs,
        ts,
        models,
        *,
        Ks,
        width,
        height,
        znear=0.01,
        zfar=100,
        uv_type="vertex",
        rot_type="mat",
        mode=["color", "depth"],
    ):
        """render a batch for textured objects
        Args:
            Rs: [b,3,3] or [b,4] tensor
            ts: [b,3] tensor
            models: list of dict, each stores
                vertex uv: {"vertices":, "faces":, "texture":, "vertex_uvs":,}
                face uv: {"vertices":, "faces":, "texture":, "face_uvs":, "face_uv_ids":,}
            Ks: [b,3,3] or [3,3]
            uv_type: `vertex` | `face`
            mode: color, depth, mask, xyz (one or more must be given)
        Returns:
            dict:
                color: bhw3
                mask: bhw
                depth: bhw
                xyz: bhw3
        """
        assert self.dib_ren.mode in ["TextureBatch"], self.dib_ren.mode
        ret = {}
        self.dib_ren.set_camera_parameters_from_RT_K(Rs, ts, Ks, height, width, near=znear, far=zfar, rot_type=rot_type)
        # points: list of [vertices, faces]
        points = [[model["vertices"][None], model["faces"].long()] for model in models]
        if uv_type == "vertex":
            uv_bxpx2 = [model["vertex_uvs"][None] for model in models]
        else:  # face uv
            uv_bxpx2 = [model["face_uvs"][None] for model in models]
            ft_fx3_list = [model["face_uv_ids"] for model in models]
        texture_bx3xthxtw = [model["texture"][None] for model in models]

        # points: list of [vertices, faces]
        # colors: list of colors
        dib_ren_im, dib_ren_prob, _, dib_ren_mask = self.dib_ren.forward(
            points=points,
            uv_bxpx2=uv_bxpx2,
            texture_bx3xthxtw=texture_bx3xthxtw,
            ft_fx3=ft_fx3_list,
        )

        ret["color"] = dib_ren_im
        ret["prob"] = dib_ren_prob.squeeze(-1)  # bhw1 -> bhw
        ret["mask"] = dib_ren_mask.squeeze(-1)  # bhw1 -> bhw

        if "depth" in mode:
            # transform xyz
            # NOTE: check whether it should be in [0, 1] (maybe need to record min, max and denormalize later)
            if not isinstance(Rs, torch.Tensor):
                Rs = torch.stack(Rs)  # list
            if rot_type == "quat":
                R_mats = quat2mat_torch(Rs)
            else:
                R_mats = Rs
            xyzs = [
                misc.transform_pts_Rt_th(model["vertices"], R_mats[_id], ts[_id])[None]
                for _id, model in enumerate(models)
            ]
            dib_ren_vc_batch = DIBRenderer(height, width, mode="VertexColorBatch")
            dib_ren_vc_batch.set_camera_parameters(self.dib_ren.camera_params)
            ren_xyzs, _, _, _ = dib_ren_vc_batch.forward(points=points, colors=xyzs)
            if "depth" in mode:
                ret["depth"] = ren_xyzs[:, :, :, 2]  # bhw

        if "xyz" in mode:  # TODO: check this
            obj_xyzs = [model["vertices"][None] for _id, model in enumerate(models)]
            dib_ren_vc_batch = DIBRenderer(height, width, mode="VertexColorBatch")
            dib_ren_vc_batch.set_camera_parameters(self.dib_ren.camera_params)
            ren_obj_xyzs, _, _, _ = dib_ren_vc_batch.forward(points=points, colors=obj_xyzs)
            ret["xyz"] = ren_obj_xyzs
        return ret  # bxhxwx3 rgb, bhw prob/mask/depth
```

## uv坐标
UV坐标就是用来在纹理图像（一张2D图片）上进行定位的横纵坐标。

UV坐标，也称为纹理坐标，是计算机图形学中用于将2D纹理图像（如一张图片）映射到3D模型表面的2D坐标系。它的核心作用是指定模型表面的每个点应该对应纹理图像上的哪个位置，从而让模型拥有丰富的颜色和细节。
关键点说明：

    坐标轴：U 和 V 是两个坐标轴，类似于 2D 平面中的 X 和 Y。U 通常代表水平方向，V 代表垂直方向。

    取值范围：UV 坐标通常归一化到 [0, 1] 区间。例如，(0,0) 对应纹理图像的左下角（或左上角，取决于约定），(1,1) 对应右上角。

    如何工作：

        3D 模型的每个顶点除了有空间位置 (X, Y, Z) 外，还会被赋予一对 UV 坐标 (U, V)。

        渲染时，对于三角形面片内的每个像素，GPU 会根据三个顶点的 UV 坐标进行插值，计算出该像素对应的 UV 坐标。

        然后从纹理图像中采样该位置的颜色，作为像素的最终颜色。

    类比：就像给一个立体模型“贴标签”，UV 坐标决定了标签的哪一部分贴在模型的哪个位置。这个过程通常需要将模型表面“展开”到 2D 平面上，称为UV 展开。

在你之前代码中的体现：

    参数 uv_bxpx2 存储的就是具体的 UV 坐标值（bxpx2 表示 batch 大小、顶点/面数、2 个坐标）。

    uv_type 决定 UV 坐标是直接绑定到每个顶点（vertex），还是通过面索引间接引用（face）。

    ft_fx3_list 在面 UV 模式下，为每个面片指定使用哪三个 UV 坐标（类似于几何中的 faces 索引顶点）。

简言之，UV 坐标是连接 3D 几何与 2D 纹理的桥梁，是实现纹理映射的基础。
