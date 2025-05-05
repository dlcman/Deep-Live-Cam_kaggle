## Run in kaggle

kaggle虽然提供gpu和tpu，而且时间很长分别是30小时/周，和20小时/周。
* "/kaggle/input"文件夹是数据集指定的文件夹，但是为只读文件（所以我在对应位置改了一下缓存路径）
* 限制挺多，不能像服务器一样直接使用，提供的终端也有很多限制。

**1. 上传资源到数据集中**
1. 将替换的头像，和待替换的视频上传到kaggle中的datasets（左侧导航栏）上，并取名为change_face。
2. 创建notebook代码，input（右侧）选择刚刚上传的change_face，将下面的代码选择性的复制进去并执行。
3. **注意：检查右侧的internet是否开启。**

**2. 环境命令:**
* 左上菜单栏Add-ons中install dependencies中复制下面命令并执行

```Jupyter Notebook
# 安装 cuDNN 8.9.x for CUDA 12.x
!apt install -y --allow-change-held-packages libcudnn8=8.9.7.*-1+cuda12.2
!ldconfig  # 刷新动态链接库

# 使用uv进行包管理（亲测快了6倍）
!pip install uv

%cd "/kaggle"
!git clone https://github.com/dlcman/Deep-Live-Cam_kaggle.git

%cd "/kaggle/Deep-Live-Cam_kaggle"
!uv venv --python=3.10
!uv pip install -p /usr/bin/python3.10 -r requirements.txt --index-strategy unsafe-best-match

print("安装结束！")
import os
os.environ['MPLBACKEND'] = 'agg'
```

**3. 修改相关的处理函数**
* 下载修复模型和面容交换模型
```python
# 下载修复模型和面容交换模型
import os
import requests

def download_file(url, save_dir, filename=None):
    file_path = os.path.join(save_dir, filename)
    print(f"Downloading {url} to {file_path}...")
    response = requests.get(url, stream=True)
    response.raise_for_status()  # 检查请求是否成功
    with open(file_path, 'wb') as file:
        for chunk in response.iter_content(chunk_size=8192):
            file.write(chunk)
    print(f"File saved to {file_path}")
    
save_dir = "/kaggle/Deep-Live-Cam_kaggle/models"
# 下载GFPGANv1.4.pth
url = "https://huggingface.co/hacksider/deep-live-cam/resolve/main/GFPGANv1.4.pth"
filename = "GFPGANv1.4.pth"
download_file(url, save_dir, filename)

# 下载inswapper_128_fp16.onnx
url = "https://huggingface.co/hacksider/deep-live-cam/resolve/main/inswapper_128_fp16.onnx"
filename = "inswapper_128_fp16.onnx"
download_file(url, save_dir, filename)
```

* 版本问题，直接修改python包文件

    先查看安装包的具体路径：
    ```bash
    !uv pip show numpy --python /usr/bin/python3.10
    ```
    再判断是否需要修改路径

```python
import re

def modify_file(file_path):
    with open(file_path, 'r', encoding='utf-8') as file:
        content = file.read()
    modified_content = re.sub(
        'from torchvision.transforms.functional_tensor import rgb_to_grayscale',
        'from torchvision.transforms.functional import rgb_to_grayscale',
        content
    )
    with open(file_path, 'w', encoding='utf-8') as file:
        file.write(modified_content)
    print(f"修改完成")
# 调用函数并传入文件路径
file_path = '/usr/local/lib/python3.10/dist-packages/basicsr/data/degradations.py'
modify_file(file_path)
```

**4. 执行命令**
```Jupyter Notebook
# 执行命令
!python3.10 run.py -s "/kaggle/input/change-face/tlz.jpg" -t "/kaggle/input/change-face/target.mkv" -o "/kaggle/working/0504_output.mp4" --execution-provider cuda --frame-processor face_swapper face_enhancer
```