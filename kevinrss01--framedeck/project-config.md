---
trigger: always_on
description: This file help to work with the remotion project and technologies (remotion.dev), it gives information about the package. This project is built on top of it. So basically, each time we work with a video and the remotion dependencies, we should use this context and file.
---


# About Remotion

Remotion is a framework that can create videos programmatically.
It is based on React.js. All output should be valid React code and be written in TypeScript.

# Project structure

A Remotion Project consists of an entry file, a Root file and any number of React component files.
A project can be scaffolded using the "npx create-video@latest --blank" command.
The entry file is usually named "src/index.ts" and looks like this:

```ts
import { registerRoot } from 'remotion';
import { Root } from './Root';

registerRoot(Root);
```

The Root file is usually named "src/Root.tsx" and looks like this:

```tsx
import { Composition } from 'remotion';
import { MyComp } from './MyComp';

export const Root: React.FC = () => {
  return (
    <>
      <Composition
        id="MyComp"
        component={MyComp}
        durationInFrames={120}
        width={1920}
        height={1080}
        fps={30}
        defaultProps={{}}
      />
    </>
  );
};
```

A `<Composition>` defines a video that can be rendered. It consists of a React "component", an "id", a "durationInFrames", a "width", a "height" and a frame rate "fps".
The default frame rate should be 30.
The default height should be 1080 and the default width should be 1920.
The default "id" should be "MyComp".
The "defaultProps" must be in the shape of the React props the "component" expects.

Inside a React "component", one can use the "useCurrentFrame()" hook to get the current frame number.
Frame numbers start at 0.

```tsx
export const MyComp: React.FC = () => {
  const frame = useCurrentFrame();
  return <div>Frame {frame}</div>;
};
```

# Component Rules

Inside a component, regular HTML and SVG tags can be returned.
There are special tags for video and audio.
Those special tags accept regular CSS styles.

If a video is included in the component it should use the "<Video>" tag.

```tsx
import { Video } from '@remotion/media';

export const MyComp: React.FC = () => {
  return (
    <div>
      <Video src="https://remotion.dev/bbb.mp4" style={{ width: '100%' }} />
    </div>
  );
};
```

Video has a "trimBefore" prop that trims the left side of a video by a number of frames.
Video has a "trimAfter" prop that limits how long a video is shown.
Video has a "volume" prop that sets the volume of the video. It accepts values between 0 and 1.

If an non-animated image is included In the component it should use the "<Img>" tag.

```tsx
import { Img } from 'remotion';

export const MyComp: React.FC = () => {
  return <Img src="https://remotion.dev/logo.png" style={{ width: '100%' }} />;
};
```

If an animated GIF is included, the "@remotion/gif" package should be installed and the "<Gif>" tag should be used.

```tsx
import { Gif } from '@remotion/gif';

export const MyComp: React.FC = () => {
  return (
    <Gif src="https://media.giphy.com/media/l0MYd5y8e1t0m/giphy.gif" style={{ width: '100%' }} />
  );
};
```

If audio is included, the "<Audio>" tag should be used.

```tsx
import { Audio } from '@remotion/media';

export const MyComp: React.FC = () => {
  return <Audio src="https://remotion.dev/audio.mp3" />;
};
```

Asset sources can be specified as either a Remote URL or an asset that is referenced from the "public/" folder of the project.
If an asset is referenced from the "public/" folder, it should be specified using the "staticFile" API from Remotion

```tsx
import { staticFile } from 'remotion';
import { Audio } from '@remotion/media';

export const MyComp: React.FC = () => {
  return <Audio src={staticFile('audio.mp3')} />;
};
```

Audio has a "trimBefore" prop that trims the left side of a audio by a number of frames.
Audio has a "trimAfter" prop that limits how long a audio is shown.
Audio has a "volume" prop that sets the volume of the audio. It accepts values between 0 and 1.

If two elements should be rendered on top of each other, they should be layered using the "AbsoluteFill" component from "remotion".

```tsx
import { AbsoluteFill } from 'remotion';

export const MyComp: React.FC = () => {
  return (
    <AbsoluteFill>
      <AbsoluteFill style={{ background: 'blue' }}>
        <div>This is in the back</div>
      </AbsoluteFill>
      <AbsoluteFill style={{ background: 'blue' }}>
        <div>This is in front</div>
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```

Any Element can be wrapped in a "Sequence" component from "remotion" to place the element later in the video.

```tsx
import { Sequence } from 'remotion';

export const MyComp: React.FC = () => {
  return (
    <Sequence from={10} durationInFrames={20}>
      <div>This only appears after 10 frames</div>
    </Sequence>
  );
};
```

A Sequence has a "from" prop that specifies the frame number where the element should appear.
The "from" prop can be negative, in which case the Sequence will start immediately but cut off the first "from" frames.

A Sequence has a "durationInFrames" prop that specifies how long the element should appear.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevinrss01/framedeck](https://github.com/kevinrss01/framedeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
