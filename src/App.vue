<template>
	<div id="napkin" v-if="napkin && napkin.lines" :class="{loaded: napkin}">
		<h1>
			<input tabindex="-1" :value="napkin.name" @keyup="updateName($event.target.value)"
			       placeholder="Title...">
			<div id="actions">
				<button id="clear" @click="napkin.lines = []">clear</button>
				<button id="share">share</button>
			</div>
		</h1>

		<div class="editor-container">
			<editor editor-id="editorA" :content="napkin.lines.join('\n')" lang="js"
			        v-on:change-content="updateLines"></editor>
		</div>

		<div id="answers">
			<div class="answer" v-for="line in napkin.lines" v-html="eval(line)"></div>
			<div id="sum">
				{{sum}}
			</div>
		</div>
	</div>
</template>


<script>
    import throttle from 'lodash.throttle'
    import sumBy from 'lodash.sumby'
    import Editor from './components/Editor';


    const throttledUpdateLines = throttle((lines) => {
        localStorage.setItem('lines', JSON.stringify(lines))
    }, 100)

    const throttledUpdateName = throttle((name) => {
        localStorage.setItem('name', name)
    }, 100)


    export default {
        name: 'app',
        data: () => ({
            napkin: {
                name: '',
                lines: []
            },
        }),


        mounted() {
            this.napkin.lines = JSON.parse(localStorage.getItem('lines')) || []
            this.napkin.name = localStorage.getItem('name')
        },

        methods: {
            updateLines(v) {
                this.napkin.lines = v.split('\n')
                throttledUpdateLines(this.napkin.lines)
            },

            updateName(name) {
                this.napkin.name = name
                throttledUpdateName(this.napkin.name)
            },

            eval(l) {
                try {
                    let prepped = l.replace(/([a-z]|[A-Z])+/g, '')
                    console.log(prepped)
                    let answ = eval(prepped)
                    if (answ != null) {
                        return answ
                    } else {
                        return "<span class='no-result'>-</span>"
                    }
                } catch (e) {
                    return "<span class='no-result'>-</span>"
                }
            },

            evalSum(l) {
                try {
                    let prepped = l.replace(/([a-z]|[A-Z])+/g, '')
                    let answ = eval(prepped)
                    if (answ != null) {
                        return answ
                    } else {
                        return 0
                    }
                } catch (e) {
                    return 0
                }
            }
        },

        computed: {
            sum() {
                return sumBy(this.napkin.lines, line => this.evalSum(line))
            }
        },

	    components: {
            Editor
	    }
    }
</script>




<style>
	@import url('https://fonts.googleapis.com/css?family=Fira+Mono');


	* {
		font-family: 'Fira Mono', Fira Code;
		box-sizing: border-box;
		--title-height: 75px;
		--border-radius: 3px;
	}
	#napkin {
		display: grid;
		grid-template-columns: 2fr 1fr;
		max-width: 800px;
		grid-template-rows: var(--title-height) auto;
		margin: auto;
		grid-gap: 0;
	}

	.editor {
		width: calc(100% - 30px) !important;
		display: flex;
		/* border-width: 1px 0 1px 1px; */
		min-height: 500px;
		/* border-style: solid; */
		/* border-color: #eee; */
	}

	.editor, #answers {

		font-size: 14px;
		line-height: 1.5 !important;
		outline: none;
		padding: 30px 2px;
		grid-row: 2;
	}

	.editor .ace_scroller {
		/* padding: 30px 0 30px 30px; */
		left: 0 !important;
	}

	#answers .no-result {
		visibility: hidden;
	}

	#answers {
		display: flex;
		background: #eee;
		border: 1px solid #eee;
		flex-direction: column;
		padding-right: 30px;
		padding-left: 30px;
		border-radius: 0 0 5px 0;
	}

	body {
		padding: 15px;
	}

	input {
		height: var(--title-height);
		font-size: 20px;
		border: 1px solid #eee;
		outline: none;
		border-radius: var(--border-radius) var(--border-radius) 0 0 !important;
		width: 100%;
	}

	#sum {
		margin-top: auto;
		font-weight: bold;
		padding-top: 14px;
		border-top: 1px solid #ccc;
	}

	input, .editor {
		padding-left: 30px;
		border-radius: 0;
		box-shadow: none;
		-webkit-appearance: none;

	}

	#napkin:not(.loaded) {
		opacity: 0;
	}

	#napkin.loaded {
		opacity: 1;
	}


	/*
	|--------------------------------------------------------------------------
	| Mobile
	|--------------------------------------------------------------------------
	*/
	@media(max-width: 600px) {
		#napkin {
			height: 100vh;

		}
		body {
			padding: 0 !important;
			margin: 0;
			height: 100vh;
		}
		input{
			font-size: 18px;
			border-top: none !important;
		}

		input, .editor, #answers {
			padding-left: 20px;
			border-left: 0;
			border-right: 0;
		}

		h1 {
			width: 100%;
			margin: 0 !important;
			grid-column: 1 / span 2;
		}

		.editor {
			margin-left: 20px !important;
			width: calc(100% - 20px) !important;
		}
		.editor-container {
			border: none !important;
		}
	}

	.editor-container {
		border-width: 1px 0 1px 1px;
		min-height: 500px;
		border-style: solid;
		border-color: #eee;
		grid-column: 1;
		border-radius: 0 0 0 var(--border-radius);
		grid-row: 2;
		border-top: none;
		padding-left: 0;
	}

	.ace_gutter-cell.ace_gutter-active-line.ace_error {
		display: none;
	}

	.ace_layer.ace_gutter-layer.ace_folding-enabled {
		display: none !important;
	}

	.ace_gutter {
		display: none;
	}

	.ace_scroller {
		left: 0;
	}

	.editor {
		margin: 30px;
	}



	input::placeholder {
		color: #a7a7a7;
	}

	h1 {
		margin-top: 0;
		border: none !important;
		grid-column: 1 / span 2;
		position: relative;
		margin-bottom: 0;
		/* display: flex; */
	}

	#actions {
		align-items: center;
		width: max-content;
		position: absolute;
		right: 0;
		top: 0;
		padding-right: 20px;
		height: 100%;
		display: flex;
		justify-content: center;
	}

	button {
		background: none;
		border: 1px solid #eee;
		height: 30px;
		border-radius: 5px;
		padding: 0 10px;
		outline: none;
	}

	#actions button {
		margin-left: 10px;
	}

	button:hover {
		background: #eee;
	}


</style>
