---
trigger: always_on
description: * this project is a drag and drop code editor
---

* this project is a drag and drop code editor
* this project uses solidjs and typescript. type safety is of the utmost importance
* there's going to be a canvas where you can drag and drop code blocks
* each code block is required to either execute an effect, or return a value
* the code blocks are reactive
* when some value changes, every single code block that depends on said value also is recomputed
* you can have target blocks which can have either like a table or a graph or just a html renderer (for now)
* i think there should be a reactivity class that is responsible for managing the reactivity of the code blocks 
* for the code blocks, we should use monaco
* for drag and drop, for now, let us use https://github.com/thisbeyond/solid-dnd (please index that)
* down the line, might use the webcontainers api to let us have any library in this editor
# here is an example of using monaco with solidjs

```typescript
import { createSignal, createEffect, onCleanup, JSX, onMount, mergeProps, on } from 'solid-js'
import * as monacoEditor from 'monaco-editor'
import loader, { Monaco } from '@monaco-editor/loader'
import { Loader } from './Loader'
import { MonacoContainer } from './MonacoContainer'
import { getOrCreateModel } from './utils'
import { LoaderParams } from './types'

const viewStates = new Map()

export interface MonacoEditorProps {
  language?: string
  value?: string
  loadingState?: JSX.Element
  class?: string
  theme?: monacoEditor.editor.BuiltinTheme | string
  path?: string
  overrideServices?: monacoEditor.editor.IEditorOverrideServices
  width?: string
  height?: string
  options?: monacoEditor.editor.IStandaloneEditorConstructionOptions
  saveViewState?: boolean
  loaderParams?: LoaderParams
  onChange?: (value: string, event: monacoEditor.editor.IModelContentChangedEvent) => void
  onMount?: (monaco: Monaco, editor: monacoEditor.editor.IStandaloneCodeEditor) => void
  onBeforeUnmount?: (monaco: Monaco, editor: monacoEditor.editor.IStandaloneCodeEditor) => void
}

export const MonacoEditor = (inputProps: MonacoEditorProps) => {
  const props = mergeProps(
    {
      theme: 'vs',
      width: '100%',
      height: '100%',
      loadingState: 'Loading…',
      saveViewState: true,
    },
    inputProps,
  )

  let containerRef: HTMLDivElement

  const [monaco, setMonaco] = createSignal<Monaco>()
  const [editor, setEditor] = createSignal<monacoEditor.editor.IStandaloneCodeEditor>()

  let abortInitialization: (() => void) | undefined
  let monacoOnChangeSubscription: any
  let isOnChangeSuppressed = false

  onMount(async () => {
    loader.config(inputProps.loaderParams ?? { monaco: monacoEditor })
    const loadMonaco = loader.init()

    abortInitialization = () => loadMonaco.cancel()

    try {
      const monaco = await loadMonaco
      const editor = createEditor(monaco)
      setMonaco(monaco)
      setEditor(editor)
      props.onMount?.(monaco, editor)

      monacoOnChangeSubscription = editor.onDidChangeModelContent(event => {
        if (!isOnChangeSuppressed) {
          props.onChange?.(editor.getValue(), event)
        }
      })
    } catch (error: any) {
      if (error?.type === 'cancelation') {
        return
      }

      console.error('Could not initialize Monaco', error)
    }
  })

  onCleanup(() => {
    const _editor = editor()
    if (!_editor) {
      abortInitialization?.()
      return
    }

    props.onBeforeUnmount?.(monaco()!, _editor)
    monacoOnChangeSubscription?.dispose()
    _editor.getModel()?.dispose()
    _editor.dispose()
  })

  createEffect(
    on(
      () => props.value,
      value => {
        const _editor = editor()
        if (!_editor || typeof value === 'undefined') {
          return
        }

        if (_editor.getOption(monaco()!.editor.EditorOption.readOnly)) {
          _editor.setValue(value)
          return
        }

        if (value !== _editor.getValue()) {
          isOnChangeSuppressed = true

          _editor.executeEdits('', [
            {
              range: _editor.getModel()!.getFullModelRange(),
              text: value,
              forceMoveMarkers: true,
            },
          ])

          _editor.pushUndoStop()
          isOnChangeSuppressed = false
        }
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.options,
      options => {
        editor()?.updateOptions(options ?? {})
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.theme,
      theme => {
        monaco()?.editor.setTheme(theme)
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.language,
      language => {
        const model = editor()?.getModel()
        if (!language || !model) {
          return
        }

        monaco()?.editor.setModelLanguage(model, language)
      },
      { defer: true },
    ),
  )

  createEffect(
    on(
      () => props.path,
      (path, prevPath) => {
        const _monaco = monaco()
        if (!_monaco) {
          return
        }

        const model = getOrCreateModel(_monaco, props.value ?? '', props.language, path)

        if (model !== editor()?.getModel()) {
          if (props.saveViewState) {
            viewStates.set(prevPath, editor()?.saveViewState())
          }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saiashirwad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
