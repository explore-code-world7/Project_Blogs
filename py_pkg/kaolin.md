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


