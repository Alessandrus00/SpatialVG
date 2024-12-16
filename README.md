<p align="center">
  <h1 align="center">Exploring the Role of Spatial Relations in Visual Grounding: A Novel Benchmark and Synthetic Pretraining </h1>
  <h3 align="center">Benchmarking spatial reasoning in visual grounding</h3>
</p>

### Overview

We are proud to introduce Spatial Visual Grounding (SpatialVG), a novel benchmark specifically designed to evaluate the spatial reasoning capabilities of modern visual grounding models.
SpatialVG images are carefully curated to contain two instances of the same object category,
which require disambiguation using a referring expression, aligning with the task requirements
of visual grounding. Unlike prior benchmarks, the referring expressions in SpatialVG are generated using a template-based approach. Each expression includes a single spatial relation that distinguishes the target object (one of the two instances) from the other, by relating it to a third, single-instance object that serves as a spatial reference. This structure allows us to probe models’ability to differentiate between similar objects based on relative spatial relationships to a third entity, effectively testing their spatial reasoning skills in a controlled context. Some examples are shown below.

_The dog next to the toilet_   |  _The person under the hairdryer_ | _The cat in front of the person_
:-------------------------:|:-------------------------:|:-------------------------:
![](docs/spatialvg_nextto.jpg)  |  ![](docs/spatialvg_under.jpg) |   ![](docs/spatialvg_infrontof.jpg)

#### Why SpatialVG?
Spatial reasoning is an essential part of human cognitive functions, underpinning the ability
to describe, understand, and interact with the environment. Natural language inherently contains
spatial relations, which are frequently used to locate objects relative to each other, such as
“above,” “to the left of,” or “behind.” These relational descriptors are critical to tasks ranging from navigation to object manipulation and are deeply embedded in everyday communication,
as people rely on spatial cues to describe scenes or guide actions. Given it importance we wanted to study it under the lens of visual grounding.

#### Discoveries
The by-relation accuracy of TransVG++ [(Deng et al., 2023)](https://doi.org/10.1109/TPAMI.2023.3296823) and VLTVG [(Yang et al., 2022)](https://doi.org/10.1109/CVPR52688.2022.00928) models trained on refCOCO/refCOCOg and evaluated on SpatialVG is shown below.

![](docs/accuracy_sprels.jpg)
Models struggle with spatial relations requiring 3D implicit reasoning (in front of and behind), vertical positioning (on, above and under) and proximity (next to). Horizontal relations (left, right) seem to be better understood, reaching significantly higher accuracy.

<img align="left" width="300" style="padding: 10px;" src="docs/spatial_reasoning_table.png"> 

**_Poor overall performance_.** The top-performing model is VLTVG trained on refCOCOg, with an overall accuracy score of 71.7%. Compared to the human ceiling, there is more than **24% gap**, meaning that there is plenty of room for improving spatial reasoning in visual grounding.
<br><br><br><br><br>

### How to run?

To reproduce the results, you should first setup VLTVG and TransVG++ folders. You can follow the original implementations available [here](https://github.com/djiajunustc/TransVG) (for VLTVG) and [here](https://github.com/yangli18/VLTVG) (for TransVG++). Then, you should perform the following operations:

For TransVG++:

1. Create the **mscoco** folder inside the ``data`` folder of TransVG++.
```bash
cd TransVG++/TransVG/data
mkdir mscoco
```
2. Copy the content of ``SpatialVG/VG format`` inside the newly created folder ``mscoco``.
3. Create the **MSCOCO2017** folder inside the ``ln_data`` folder of TransVG++.
```bash
cd TransVG++/TransVG/ln_data
mkdir MSCOCO2017
```
4. Put all the images present in the train and validation splits of MSCOCO 2017 inside the new folder ``MSCOCO2017``. You can download them [here](https://cocodataset.org/#download).
5. Add the following entry to the dictionary called ``SUPPORTED_DATASETS`` inside the file ``TransVG++/TransVG/datasets/data_loader.py``:
```python
'mscoco':{
    'splits': ('spatialvg-all','spatialvg-left','spatialvg-right','spatialvg-front','spatialvg-behind','spatialvg-on','spatialvg-under','spatialvg-next','spatialvg-above'),
    'params': {'dataset': 'SpatialVG', 'split_by': 'Alessandrus00'}
}
```
6. Add the following **elif** statement in the same python file:

```python
elif self.dataset == 'mscoco':
    self.dataset_root = osp.join(self.data_root, 'MSCOCO2017')
    self.im_dir = osp.join(self.dataset_root)
```
before the **else** statement in the **init** function of the **TransVGDataset** class:

```python
else:   ## refcoco, etc.
    self.dataset_root = osp.join(self.data_root, 'other')
    self.im_dir = osp.join(
        self.dataset_root, 'images', 'mscoco', 'images', 'train2014')
    self.split_dir = osp.join(self.dataset_root, 'splits')

```

#### TransVG++: evaluate the models on the desired SpatialVG split
Make sure you have the models trained on refCOCO/refCOCOg and stored under ``TransVG++/TransVG/outputs``. Change the **eval_model** path according to the name you chose.
``` bash
cd TransVG
python -m torch.distributed.launch --nproc_per_node=1 --use_env --master_port 47770 \
    eval.py \
    --batch_size 16 \
    --vit_model tiny \
    --bert_model bert-base-uncased \
    --max_query_len 20 \
    --dataset mscoco \
    --eval_set spatialvg-all \
    --reg_out_type reg_token \
    --language_modulation cross_attn \
    --without_visual_mask \
    --eval_model outputs/transvg++_refcoco_vit_tiny
```

For VLTVG the procedure is similar. The only difference is the naming of the folders. The content of ``SpatialVG/VG format`` must be put inside ``VLTVG/split/data/mscoco`` and the ``MSCOCO2017`` inside ``VLTVG/data``. The additions of point number 5 and 6 must be done inside the file ``VLTVG/datasets/dataset.py``.

#### VLTVG: evaluate the models on the desired SpatialVG split
```bash
cd VLTVG
python -m torch.distributed.launch --nproc_per_node=1 --use_env \
    test.py \
    --config configs/VLTVG_R50_unc.py \
    --checkpoint work_dirs/vltvg_refcoco_r50/checkpoint_best_acc.pth \
    --batch_size_test 16 \
    --dataset mscoco \
    --test_split spatialvg-all \
```

### Trained models

To reproduce our results, we provide the trained VLTVG and TransVG++ models [here](https://drive.google.com/drive/folders/1gmipselyfDHZle_sGAKd4gQhWC7uKbEp?usp=sharing). The folders must be copied to ``TransVG++/TransVG/outputs`` and ``VLTVG/work_dirs``, accordingly.