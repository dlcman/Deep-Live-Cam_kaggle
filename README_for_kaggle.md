## Run in kaggle

kaggle虽然提供gpu和tpu，而且时间很长分别是30小时/周，和20小时/周。
* "/kaggle/input"文件夹是数据集指定的文件夹，但是为只读文件
* 限制挺多，不能像服务器一样直接使用，提供的终端也有很多限制。
**项目下的run_in_kaggle.py是运行在kaggle的代码，可以直接运行。**



**1. 上传资源到数据集中**
1. 将替换的头像，和待替换的视频上传到kaggle中的datasets（左侧导航栏）上，并取名为change_face。
2. 创建notebook代码，input（右侧）选择刚刚上传的change_face，将下面的代码选择性的复制进去并执行。
3. **注意：检查右侧的internet是否开启。**

**2. 克隆当前项目**
```bash
# 克隆项目
%cd "/kaggle"
!git clone https://github.com/dlcman/Deep-Live-Cam_kaggle.git
# 进入项目
%cd "/kaggle/Deep-Live-Cam_kaggle"
!pip install -r requirements.txt
# 执行命令
!python3.10 run.py -s "/kaggle/input/change-face/tlz.jpg" -t "/kaggle/input/change-face/0504-1_x264.mp4" -o "/kaggle/working/0504_output.mp4" --execution-provider cuda
```
