<template>
	<div :id="editorId" class="editor" style="width: 100%; height: 100%;"></div>
</template>

<script>
    export default {
        name: "Editor",
        props: ['editorId', 'content', 'lang', 'theme'],
        data () {
            return {
                editor: Object,
                beforeContent: ''
            }
        },
        watch: {
            'content' (value) {
                if (this.beforeContent !== value) {
                    this.editor.setValue(value, 1)
                }
            }
        },
        mounted () {
            const lang = this.lang || 'text'
            const theme = this.theme || 'textmate'

            this.editor = window.ace.edit(this.editorId)
            this.editor.setValue(this.content, 1)

            // mode-xxx.js or theme-xxx.jsãŒã‚ã‚‹å ´åˆã®ã¿æœ‰åŠ¹
            this.editor.getSession().setMode(`ace/mode/javascript`)
            this.editor.setTheme(`ace/theme/${theme}`)


            this.editor.on('change', () => {
                this.beforeContent = this.editor.getValue()
                this.$emit('change-content', this.editor.getValue())
            })
        }
    }
</script>

<style scoped>

</style>