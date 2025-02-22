<script>
export default {
  name: 'CodeEditor',

  model: {
    event: 'change'
  },

  props: {
    original: String,
    value: {
      type: String,
      required: true
    },
    theme: {
      type: String,
      default: 'vs'
    },
    language: String,
    options: Object,
    amdRequire: {
      type: Function
    },
    diffEditor: {
      type: Boolean,
      default: false
    },
    readonly: {
      type: Boolean,
      default: false
    }
  },

  watch: {
    options: {
      deep: true,
      handler (options) {
        if (this.editor) {
          const editor = this.getModifiedEditor();
          editor.updateOptions(options);
        }
      }
    },

    value (newValue) {
      if (this.editor) {
        const editor = this.getModifiedEditor();
        if (newValue !== editor.getValue()) {
          editor.setValue(newValue);
        }
      }
    },

    original (newValue) {
      if (this.editor && this.diffEditor) {
        const editor = this.getOriginalEditor();
        if (newValue !== editor.getValue()) {
          editor.setValue(newValue);
        }
      }
    },

    language (newVal) {
      if (this.editor) {
        const editor = this.getModifiedEditor();
        this.monaco.editor.setModelLanguage(editor.getModel(), newVal);
      }
    },

    theme (newVal) {
      if (this.editor) {
        this.monaco.editor.setTheme(newVal);
      }
    }
  },

  mounted () {
    if (this.amdRequire) {
      this.amdRequire(['vs/editor/editor.main'], () => {
        this.monaco = window.monaco;
        this.$nextTick(() => {
          this.initMonaco(window.monaco);
        });
      });
    } else {
      // ESM format so it can't be resolved by commonjs `require` in eslint
      // eslint-disable-next-line import/no-unresolved
      const monaco = require('monaco-editor');
      this.monaco = monaco;
      this.$nextTick(() => {
        this.initMonaco(monaco);
      });
    }
  },

  beforeDestroy () {
    this.editor && this.editor.dispose();
  },

  methods: {
    initMonaco (monaco) {
      this.$emit('editorWillMount', this.monaco);

      const options = Object.assign(
        {
          value: this.value,
          theme: this.theme,
          language: this.language,
          readOnly: this.readonly,
          fontSize: 14,
          glyphMargin: true,
          scrollBeyondLastLine: false,
          // 自动调整大小，可能会影响性能
          automaticLayout: true
        },
        this.options
      );

      if (this.diffEditor) {
        this.editor = monaco.editor.createDiffEditor(this.$el, options);
        const originalModel = monaco.editor.createModel(
          this.original,
          this.language
        );
        const modifiedModel = monaco.editor.createModel(
          this.value,
          this.language
        );
        this.editor.setModel({
          original: originalModel,
          modified: modifiedModel
        });
      } else {
        this.editor = monaco.editor.create(this.$el, options);
      }

      // @event `change`
      const editor = this.getModifiedEditor();
      editor.onDidChangeModelContent((event) => {
        const value = editor.getValue();
        if (this.value !== value) {
          this.$emit('change', value, event);
        }
      });

      this.$emit('editorDidMount', this.editor);
    },

    getMonaco () {
      return this.monaco;
    },

    getEditor () {
      return this.editor;
    },

    getModifiedEditor () {
      return this.diffEditor ? this.editor.getModifiedEditor() : this.editor;
    },

    getOriginalEditor () {
      return this.diffEditor ? this.editor.getOriginalEditor() : this.editor;
    },

    focus () {
      this.editor.focus();
    },

    getMarkers () {
      const uri = this.editor._modelData.model.uri;
      return this.monaco.editor.getModelMarkers({ resource: uri });
    }
  },

  render (h) {
    return h('div');
  }
};
</script>
