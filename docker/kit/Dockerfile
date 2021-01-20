ARG ROS_DISTRO=melodic
FROM osrf/ros:${ROS_DISTRO}-desktop-full


# Dockerのビルドに必要な環境変数の設定
#------------------------------------------------


# Dockerのビルドに必要なツールのインストール
#------------------------------------------------
RUN apt-get update -q && apt-get upgrade -yq && apt-get install -y --no-install-recommends \
        libglib2.0-dev-bin \
        dh-python \
        apt-utils \
        python-pip \
        git \
        lsb-release \
        tzdata \
    && pip install --upgrade pip \
    && rm -rf /var/lib/apt/lists/*


# 依存ツール/ライブラリのインストール
#------------------------------------------------
RUN apt-get update -q && apt-get install -y --no-install-recommends \
        gnome-terminal \
        dbus-x11 \
        at-spi2-core \
        python-tk \
        python-pil.imagetk \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update -q && apt-get install -y --no-install-recommends \
        ros-${ROS_DISTRO}-turtlebot3 \
        ros-${ROS_DISTRO}-turtlebot3-msgs \
        ros-${ROS_DISTRO}-aruco-ros \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update -q && apt-get install -y --no-install-recommends \
        ros-${ROS_DISTRO}-turtlebot3-simulations \
    && rm -rf /var/lib/apt/lists/*

RUN yes | pip install \
    requests \
    flask


# コンテナ内で使いたいツール等をインストール
#------------------------------------------------
RUN apt-get update -q && apt-get install -y --no-install-recommends \
        vim \
        tmux \
    && rm -rf /var/lib/apt/lists/*


# 開発者ユーザーの追加
#------------------------------------------------
ARG USERNAME=developer
RUN useradd --create-home --home-dir /home/${USERNAME} --shell /bin/bash --user-group --groups adm,sudo,video,audio ${USERNAME} \
    && echo ${USERNAME}:${USERNAME} | chpasswd \
    && echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER ${USERNAME}


# ROSの初期化
#------------------------------------------------
RUN test -f /etc/ros/rosdep/sources.list.d/20-default.list && \
    sudo rm /etc/ros/rosdep/sources.list.d/20-default.list
RUN sudo rosdep init && \
    rosdep update


# 補助スクリプトのコピー
#------------------------------------------------
COPY scripts /home/${USERNAME}/scripts
RUN sudo chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/scripts \
    && chmod -R +x /home/${USERNAME}/scripts


# ワークスペース作成
#------------------------------------------------
ARG ROBOCON_WS=/home/${USERNAME}/catkin_ws
RUN mkdir -p ${ROBOCON_WS}/src


# コンテナ起動時の設定
#------------------------------------------------
# 永続化したい設定をbashrcに追加
COPY templates/bashrc_config /home/${USERNAME}/bashrc_config
RUN cat /home/${USERNAME}/bashrc_config | sed "s@{ROBOCON_WS}@$ROBOCON_WS@g" >> ~/.bashrc \
    && rm /home/${USERNAME}/bashrc_config

# 必要なファイルを展開
COPY templates/.ignition /home/${USERNAME}/.ignition
RUN sudo chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.ignition

# 環境変数の設定
ENV TZ Asia/Tokyo

# 初期ディレクトリの指定
WORKDIR ${ROBOCON_WS}

# 起動時の実行コマンドの指定
COPY  entrypoint.sh /entrypoint.sh
RUN sudo chown ${USERNAME}:${USERNAME} /entrypoint.sh \
    && chmod +x /entrypoint.sh \
    && sudo sed -i'' -e "s/^\(DEVELOPER_NAME\)=.*$/\1=${USERNAME}/g" /entrypoint.sh
ENTRYPOINT [ "/entrypoint.sh" ]
CMD [ "bash" ]
