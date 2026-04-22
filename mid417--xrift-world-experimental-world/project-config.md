---
trigger: always_on
description: このドキュメントは、AI（Claude、ChatGPT、Cursor等）がXRiftワールドを作成・修正する際のガイドです。
---

# XRift World Template - AI ガイド

このドキュメントは、AI（Claude、ChatGPT、Cursor等）がXRiftワールドを作成・修正する際のガイドです。

---

## 最重要ルール（必ず守ること）

1. **アセット読み込みは必ず `useXRift()` の `baseUrl` を使用**
2. **アセットファイルは `public/` ディレクトリに配置**
3. **`baseUrl` は末尾に `/` を含むため、`${baseUrl}path` で結合**（`${baseUrl}/path` は NG）

```typescript
// ✅ 正しい
const { baseUrl } = useXRift()
const model = useGLTF(`${baseUrl}models/robot.glb`)

// ❌ 間違い
const model = useGLTF('/models/robot.glb')           // 絶対パス NG
const model = useGLTF(`${baseUrl}/models/robot.glb`) // 余分な / NG
```

---

## プロジェクト概要

- **用途**: XRiftプラットフォーム用WebXRワールド
- **技術**: React Three Fiber + Rapier物理エンジン + Module Federation
- **動作**: CDNにアップロード後、フロントエンドから動的ロード

---

## @xrift/world-components API

### フック

| フック | 用途 | 戻り値 |
|--------|------|--------|
| `useXRift()` | アセットURL取得 | `{ baseUrl: string }` |
| `useInstanceState(key, initial)` | 全ユーザー間で状態同期 | `[value, setValue]` |
| `useUsers()` | ユーザー情報・位置取得 | `{ localUser, remoteUsers, getMovement, getLocalMovement }` |
| `useSpawnPoint()` | スポーン地点取得 | `{ position, yaw }` |
| `useScreenShareContext()` | 画面共有状態 | `{ videoElement, isSharing, startScreenShare, stopScreenShare }` |

### コンポーネント

| コンポーネント | 用途 | 主要Props |
|---------------|------|-----------|
| `Interactable` | クリック可能オブジェクト | `id`(必須), `onInteract`(必須), `interactionText`, `enabled` |
| `SpawnPoint` | プレイヤー出現地点 | `position`, `yaw`(0-360度) |
| `Mirror` | 反射面 | `position`, `rotation`, `size`, `color`, `textureResolution` |
| `VideoScreen` | 動画再生 | `id`(必須), `position`, `url`, `playing`, `sync` |
| `ScreenShareDisplay` | 画面共有表示 | `id`(必須), `position`, `rotation`, `width` |

---

## コード生成テンプレート

### GLBモデル読み込み

```typescript
import { useXRift } from '@xrift/world-components'
import { useGLTF } from '@react-three/drei'
import { RigidBody } from '@react-three/rapier'

export const MyModel = () => {
  const { baseUrl } = useXRift()
  const { scene } = useGLTF(`${baseUrl}models/model.glb`)

  return (
    <RigidBody type="fixed">
      <primitive object={scene} castShadow receiveShadow />
    </RigidBody>
  )
}
```

### テクスチャ読み込み

```typescript
import { useXRift } from '@xrift/world-components'
import { useTexture } from '@react-three/drei'

export const TexturedMesh = () => {
  const { baseUrl } = useXRift()
  const texture = useTexture(`${baseUrl}textures/albedo.png`)

  return (
    <mesh>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial map={texture} />
    </mesh>
  )
}
```

### 複数テクスチャ（PBR）

```typescript
import { useXRift } from '@xrift/world-components'
import { useTexture } from '@react-three/drei'

export const PBRMaterial = () => {
  const { baseUrl } = useXRift()
  const [albedo, normal, roughness] = useTexture([
    `${baseUrl}textures/albedo.png`,
    `${baseUrl}textures/normal.png`,
    `${baseUrl}textures/roughness.png`,
  ])

  return (
    <meshStandardMaterial
      map={albedo}
      normalMap={normal}
      roughnessMap={roughness}
    />
  )
}
```

### Skybox（360度パノラマ背景）

```typescript
import { useXRift } from '@xrift/world-components'
import { useTexture } from '@react-three/drei'
import { BackSide } from 'three'

export const Skybox = ({ radius = 500 }) => {
  const { baseUrl } = useXRift()
  const texture = useTexture(`${baseUrl}skybox.jpg`)

  return (
    <mesh>
      <sphereGeometry args={[radius, 60, 40]} />
      <meshBasicMaterial map={texture} side={BackSide} />
    </mesh>
  )
}
```

### インタラクション + 状態同期

```typescript
import { Interactable, useInstanceState } from '@xrift/world-components'

export const InteractiveButton = ({ id }: { id: string }) => {
  // useInstanceState: 全ユーザー間で同期される状態
  const [clickCount, setClickCount] = useInstanceState(`${id}-count`, 0)

  return (
    <Interactable
      id={id}
      onInteract={() => setClickCount((prev) => prev + 1)}
      interactionText={`クリック回数: ${clickCount}`}
    >
      <mesh>
        <boxGeometry args={[1, 1, 0.2]} />
        <meshStandardMaterial color={clickCount > 0 ? 'green' : 'gray'} />
      </mesh>
    </Interactable>
  )
}
```

### アニメーション（useFrame）

```typescript
import { useRef } from 'react'
import { useFrame } from '@react-three/fiber'
import type { Mesh } from 'three'

export const RotatingCube = ({ speed = 1 }) => {
  const meshRef = useRef<Mesh>(null)

  useFrame((_, delta) => {
    if (meshRef.current) {
      meshRef.current.rotation.y += delta * speed
    }
  })

  return (
    <mesh ref={meshRef}>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="hotpink" />
    </mesh>
  )
}
```

### ユーザー位置追跡

```typescript
import { useFrame } from '@react-three/fiber'
import { useUsers } from '@xrift/world-components'

export const UserTracker = () => {
  const { remoteUsers, getMovement, getLocalMovement } = useUsers()

  useFrame(() => {
    // 自分の位置
    const myMovement = getLocalMovement()
    console.log('My position:', myMovement.position)

    // 他ユーザーの位置
    remoteUsers.forEach((user) => {
      const movement = getMovement(user.socketId)
      if (movement) {
        console.log(`${user.displayName}:`, movement.position)
      }
    })
  })

  return null
}
```

---

## 型定義

### User

```typescript
interface User {
  id: string           // 認証ユーザーID
  socketId: string     // ソケット接続ID
  displayName: string  // 表示名
  avatarUrl: string | null

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mid417) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
