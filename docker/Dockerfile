FROM nvidia/cuda:10.2-cudnn7-devel-ubuntu16.04
USER root
ENV DEBIAN_FRONTEND=noninteractive \
	LANG=C.UTF-8 \
	LC_ALL=C.UTF-8 \
	PATH=/opt/conda/bin:$PATH \
	NOTEBOOK_DIR=/home/jovyan \
	NOTEBOOK_IP=0.0.0.0 \
	NOTEBOOK_PORT=8888 \
	NB_PREFIX=''


RUN apt-get update -qq \
 && apt-get upgrade -y \
 && apt-get install --no-install-recommends -y \
	curl grep sed dpkg wget bzip2 ca-certificates vim \
	libglib2.0-0 libxext6 libsm6 libxrender1 \
	git mercurial subversion \
	# opencv3
	libgtk2.0-0 \
# anaconda install
 && echo 'export PATH=/opt/conda/bin:$PATH' > /etc/profile.d/conda.sh \
 #&& ANACONDA_INSTALL_SCRIPT=`curl https://repo.continuum.io/archive/ | grep '>Anaconda3.*-Linux-x86_64.sh<' | sed -e 's/.*>\(.*.sh\)<\/.*/\1/g' | sort -r | head -1` \
 && ANACONDA_INSTALL_SCRIPT='Anaconda3-4.2.0-Linux-x86_64.sh' \
 && wget --quiet https://repo.continuum.io/archive/$ANACONDA_INSTALL_SCRIPT -O ~/anaconda.sh \
 && /bin/bash ~/anaconda.sh -b -p /opt/conda \
 && rm ~/anaconda.sh \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*
 
# ModuleNotFoundError: No module named 'conda'
# https://github.com/conda/conda/issues/9004
# conda install conda=4.6.14
# conda update --all

# tensorrt dose not support python3.6; have to downgrade python to 3.5
RUN python --version \
 && conda --version \
 && conda install conda \
 && conda --version \
 && conda info \
 && python --version \
 && pip --version \
 && conda update --all \
 && conda install -y --quiet jupyter Pillow \
 && conda config --add channels conda-forge \
	# opencv3 install
 && conda install -c menpo opencv3 \
 && conda clean -tipsy

# Install tensorflow-gpu
RUN conda --version \
 && python --version \
 && pip --version \
 && pip install apache-beam \
	tensorflow-gpu==1.14.0 \
	tensorflow-transform==0.13.0 \
	tensorflow_model_analysis==0.13.0 \
	tensorflow_data_validation==0.13.0 \
	keras \
	--ignore-installed

# tini install
ENV TINI_VERSION=v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /usr/bin/tini
RUN chmod +x /usr/bin/tini

# pytorch install
RUN conda install -y --quiet \
	numpy pyyaml mkl mkl-include setuptools cmake cffi typing \
 && conda install -y --quiet -c mingfeima \
	mkldnn \
 && conda install -y --quiet -c pytorch \
	torchvision \
	magma-cuda91 \
	cuda91 \
	'pytorch=1.3.0'

# Install Kale python package globally
# Install Kale JupyterLab extension
# pip install kfp --upgrade --user
# conda install -c conda-forge nodejs
RUN     conda install -y --quiet -c conda-forge nodejs && \
	echo 'python' && python --version && \
	echo 'conda' && conda --version && \
	echo 'jupyter' && jupyter --version && \
	echo 'node' && node --version && \
	echo 'npm' && npm --version && \
	echo 'notebook' && jupyter notebook --version && \
#	pip install kubeflow-kale kfp
	pip install kfp && \
	git clone https://github.com/jiapinai/kale.git && \
	cd kale && \
	sed -i 's/3.6.0/3.5.0/' setup.py && \
	python setup.py install && \
	cd .. && \
	rm -rf kale

#Jupyter Notebook代码自动补全
# -i https://pypi.mirrors.ustc.edu.cn/simple
RUN pip install  \
	jupyter_contrib_nbextensions jupyter_nbextensions_configurator autopep8 && \
	jupyter contrib nbextension install && \
	jupyter nbextensions_configurator enable
ADD conf/notebook.json /root/.jupyter/nbconfig/notebook.json

RUN pip install jupyterlab==1.2.6 && \
	jupyter labextension install kubeflow-kale-launcher
RUN pip uninstall numpy -y; \
	pip uninstall numpy -y; \
	pip uninstall numpy -y; \
	pip uninstall numpy -y; \
	pip uninstall numpy -y; \
	pip install numpy==1.17.5

# 不再需要安装 tensorrt, 因为不需要吧pytorch转换成tensorrt格式
# tensorrt install
# https://docs.nvidia.com/deeplearning/sdk/tensorrt-install-guide/index.html
# tensorrt available versions
# https://developer.download.nvidia.cn/compute/machine-learning/repos/rhel7/x86_64/
# ENV version='7.0.0-1.cuda9.0'
# ENV version='7.0.0-1+cuda10.2'
# RUN apt-get update -qq \
#  && apt-get install -y \
# 	libnvinfer7=${version} \
# 	libnvonnxparsers7=${version} \
# 	libnvparsers7=${version} \
# 	libnvinfer-plugin7=${version} \
# 	libnvinfer-dev=${version} \
# 	libnvonnxparsers-dev=${version} \
# 	libnvparsers-dev=${version} \
# 	libnvinfer-plugin-dev=${version} \
# 	python-libnvinfer=${version} \
# 	python3-libnvinfer=${version} \
#  && apt-mark hold \
# 	libnvinfer7 libnvonnxparsers7 libnvparsers7 libnvinfer-plugin7 \
# 	libnvinfer-dev libnvonnxparsers-dev libnvparsers-dev \
# 	libnvinfer-plugin-dev python-libnvinfer python3-libnvinfer \
#  && apt-get clean \
#  && rm -rf /var/lib/apt/lists/*
# 
# # this's kind of important to make sure the lib is install for the python installed by conda
# RUN ln -s /usr/lib/python3.5/dist-packages/tensorrt /opt/conda/lib/python3.5/site-packages/
# 
# # install torch2trt
# # https://github.com/NVIDIA-AI-IOT/torch2trt
# RUN git clone https://github.com/NVIDIA-AI-IOT/torch2trt && \
# 	cd torch2trt && \
# 	python setup.py install && \
# 	cd .. && \
# 	rm -rf torch2trt

# Install kubectl
RUN curl -L http://public.jiapinai.com/files_manager/1580132112966_kubectl-v1.17.2 -o kubectl \
	&& chmod +x ./kubectl \
	&& mv ./kubectl /usr/local/bin \
	&& wget http://public.jiapinai.com/files_manager/1581330474228_v1.10.0_ubuntu1604.clients.tar.gz -o trt-clients.tar.gz \
	&& echo 'install tensorrt inference server client' \
	&& tar xzf trt-clients.tar.gz \
	&& rm -f trt-clients.tar.gz \
	&& pip install ./v1.10.0_ubuntu1604.clients/python/tensorrtserver-1.10.0-py2.py3-none-linux_x86_64.whl \
	&& rm -rf ./v1.10.0_ubuntu1604.clients

ENV PUBLIC_URL='/nni/'
RUN pip install --upgrade nni

ENTRYPOINT [ "/usr/bin/tini", "--" ]

# https://www.kubeflow.org/docs/notebooks/custom-notebook/
CMD jupyter notebook \
	--config=/root/.jupyter/jupyter_notebook_config.json \
	--notebook-dir=${NOTEBOOK_DIR} \
	--ip=${NOTEBOOK_IP} \
	--port=${NOTEBOOK_PORT} \
	--NotebookApp.token='' \
	--NotebookApp.password='' \
	--NotebookApp.allow_origin='*' \
	--NotebookApp.base_url=${NB_PREFIX} \
	--no-browser \
	--allow-root
