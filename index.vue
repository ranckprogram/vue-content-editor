<template>
    <div ref="contentEditor" v-html="msg" @focus="handleFocus"></div>
</template>

<script>
export default {
    name: 'ContentEditor',
    props: {
        message: {
            type: String,
            default: ''
        }
    },
    data () {
        return {
            msg: ''
        }
    },
    mounted() {
        this.initStyle()
        this.addEvent()
    },
    methods: {
        initStyle() {
            const content = this.$refs.contentEditor
            content.style.width = '100%'
            content.style.height = '100%'
            content.style.outline = 'none'
            content.style['-webkit-user-select'] = 'auto; margin: 0;'
            content.setAttribute('contenteditable', true)
        },
        addEvent() {
            const content = this.$refs.contentEditor
            const self = this
            let isCN = false
            let noteMax = 50
            let oldDate
            content.addEventListener('compositionstart', function () {
                isCN = true
            })
            content.addEventListener('compositionend', function () {
                isCN = false
            })

            let txtAnchorOffset, txtFocusOffset

            content.addEventListener("textInput", function (event) {
                const _sel = document.getSelection()
                txtAnchorOffset = _sel.anchorOffset
                txtFocusOffset = _sel.focusOffset
                //必须加上isCN的判断，否则获取不到正确的光标数据
                if (!isCN && this.textContent.length >= noteMax) {
                    event.preventDefault()
                }
            })
            content.addEventListener("input", function (event) {
                setTimeout(function () {
                    if (!isCN) {
                        const _this = content
                        if (_this.textContent.length > noteMax) {
                            var data = _this.textContent
                            oldDate = data.slice(0, txtAnchorOffset) + data.slice(txtFocusOffset, data.length);
                            _this.textContent = oldDate.slice(0, noteMax)
                            document.getSelection().collapse(_this.firstChild, txtAnchorOffset)
                        }
                        self.$emit('onChange', _this.innerHTML)
                    }
                }, 0)
            })
        },
        focus () {
            const el = this.$refs.contentEditor
            el.focus()
        },
        handleFocus () {
            this.$emit('onFocus')
        },
        keepLastIndex() {
            const obj = this.$refs.contentEditor
            if (window.getSelection) {
                obj.focus()
                const range = window.getSelection()
                range.selectAllChildren(obj)
                range.collapseToEnd()
            } else if (document.selection) {
                const range = document.selection.createRange()
                range.moveToElementText(obj)
                range.collapse(false)
                range.select()
            }
        }
    },
    watch: {
        message (value) {
            this.msg = value
            this.$nextTick().then(() => {
                this.keepLastIndex()
            })
            
        }
    }
}
</script>
