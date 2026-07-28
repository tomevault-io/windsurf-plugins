---
trigger: always_on
description: In Reinforcement learning, the Agent is the entity to train to solve an environment. It's able to interact with the environment: observe, take actions, and learn through trial and error.
---

(agent_page)=

# How to use an Agent
In Reinforcement learning, the Agent is the entity to train to solve an environment. It's able to interact with the environment: observe, take actions, and learn through trial and error.
In rlberry, you can use existing Agent, or create your own custom Agent. You can find the API [here](/api) and [here](rlberry.agents.Agent) .


## Use rlberry Agent
An agent needs an environment to train. We'll use the same environment as in the [environment](environment_page) section of the user guide.
("Chain" environment from "[rlberry-scool](https://github.com/rlberry-py/rlberry-scool)")

### without agent
```python
from rlberry_scool.envs.finite import Chain

env = Chain(10, 0.1)
env.enable_rendering()
for tt in range(50):
    env.step(env.action_space.sample())
env.render(loop=False)

# env.save_video is only available for rlberry envs and custom env (with 'RenderInterface' as parent class)
video = env.save_video("_agent_page_chain1.mp4")
env.close()
```
</br>

<video controls="controls" style="max-width: 600px;">
   <source src="../../user_guide_video/_agent_page_chain1.mp4" type="video/mp4">
</video>

If we use random actions on this environment, we don't have good results (the cross don't go to the right)

### With agent

With the same environment, we will use an Agent to choose the actions instead of random actions.
For this example, you can use "ValueIterationAgent" Agent from "[rlberry-scool](https://github.com/rlberry-py/rlberry-scool)"

```python
from rlberry_scool.envs.finite import Chain
from rlberry_scool.agents.dynprog import ValueIterationAgent

env = Chain(10, 0.1)  # same env
agent = ValueIterationAgent(env, gamma=0.95)  # creation of the agent
info = agent.fit()  # Agent's training   (ValueIteration don't use budget)
print(info)

# test the trained agent
env.enable_rendering()
observation, info = env.reset()
for tt in range(50):
    action = agent.policy(
        observation
    )  # use the agent's policy to choose the next action
    observation, reward, terminated, truncated, info = env.step(action)  # do the action
    done = terminated or truncated
    if done:
        break  # stop if the environement is done
env.render(loop=False)

# env.save_video is only available for rlberry envs and custom env (with 'RenderInterface' as parent class)
video = env.save_video("_agent_page_chain2.mp4")
env.close()
```

```none
{'n_iterations': 269, 'precision': 1e-06}
  pg.display.set_mode(display, DOUBLEBUF | OPENGL)
  _ = pg.display.set_mode(display, DOUBLEBUF | OPENGL)
ffmpeg version 4.4.2-0ubuntu0.22.04.1 Copyright (c) 2000-2021 the FFmpeg developers
  built with gcc 11 (Ubuntu 11.2.0-19ubuntu1)
  configuration: --prefix=/usr --extra-version=0ubuntu0.22.04.1 --toolchain=hardened --libdir=/usr/lib/x86_64-linux-gnu --incdir=/usr/include/x86_64-linux-gnu --arch=amd64 --enable-gpl --disable-stripping --enable-gnutls --enable-ladspa --enable-libaom --enable-libass --enable-libbluray --enable-libbs2b --enable-libcaca --enable-libcdio --enable-libcodec2 --enable-libdav1d --enable-libflite --enable-libfontconfig --enable-libfreetype --enable-libfribidi --enable-libgme --enable-libgsm --enable-libjack --enable-libmp3lame --enable-libmysofa --enable-libopenjpeg --enable-libopenmpt --enable-libopus --enable-libpulse --enable-librabbitmq --enable-librubberband --enable-libshine --enable-libsnappy --enable-libsoxr --enable-libspeex --enable-libsrt --enable-libssh --enable-libtheora --enable-libtwolame --enable-libvidstab --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx265 --enable-libxml2 --enable-libxvid --enable-libzimg --enable-libzmq --enable-libzvbi --enable-lv2 --enable-omx --enable-openal --enable-opencl --enable-opengl --enable-sdl2 --enable-pocketsphinx --enable-librsvg --enable-libmfx --enable-libdc1394 --enable-libdrm --enable-libiec61883 --enable-chromaprint --enable-frei0r --enable-libx264 --enable-shared
  libavutil      56. 70.100 / 56. 70.100
  libavcodec     58.134.100 / 58.134.100
  libavformat    58. 76.100 / 58. 76.100
  libavdevice    58. 13.100 / 58. 13.100
  libavfilter     7.110.100 /  7.110.100
  libswscale      5.  9.100 /  5.  9.100
  libswresample   3.  9.100 /  3.  9.100
  libpostproc    55.  9.100 / 55.  9.100
Input #0, rawvideo, from 'pipe:':
  Duration: N/A, start: 0.000000, bitrate: 38400 kb/s
  Stream #0:0: Video: rawvideo (RGB[24] / 0x18424752), rgb24, 800x80, 38400 kb/s, 25 tbr, 25 tbn, 25 tbc
Stream mapping:
  Stream #0:0 -> #0:0 (rawvideo (native) -> h264 (libx264))
[libx264 @ 0x5570932967c0] using cpu capabilities: MMX2 SSE2Fast SSSE3 SSE4.2 AVX FMA3 BMI2 AVX2 AVX512
[libx264 @ 0x5570932967c0] profile High, level 1.3, 4:2:0, 8-bit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rlberry-py/rlberry](https://github.com/rlberry-py/rlberry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
