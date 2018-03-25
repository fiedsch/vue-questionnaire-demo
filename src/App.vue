<template>
    <div id="app">

        <div class="question-wrapper">
            <span class="question-name">{{ currentquestion.name }}</span>
            <span class="question-ask">{{ currentquestion.ask }}</span>
            <span v-if="currentquestion.tip" class="question-tip">{{ currentquestion.tip }}</span>

            <transition name="disco2">
                <ul class="singleselect" v-if="currentquestion.type==='singleselect' && currentquestion.responses">
                    <li v-for="response in currentquestion.responses" :class="{'dk':response.dk}"
                        @click="setresponse(currentquestion.name, response.code)"
                    >
                        <input type="radio"
                               :name="currentquestion.name"
                               :id="currentquestion.name+response.code"
                               :value="response.code"
                               v-model="responses[currentquestion.name]"
                        >
                        <label
                                :for="currentquestion.name+response.code"
                                :class="{'selected': response.code==responses[currentquestion.name]}"
                        >{{ response.label }}</label>
                    </li>
                </ul>
            </transition>

            <transition name="disco">
                <div class="drag-wrapper" v-show="currentquestion.type==='draganddrop'">
                    <draggable element="ul"
                               v-model="dragdata"
                               :options="{group:'draggableelements'}"
                               class="draggable"
                               @update="updateDraggable"
                               @add="updateDraggable"
                               @remove="updateDraggable"
                    >
                        <li class="draggable-element" v-for="element in dragdata" :key="element.id">
                            {{ element.name }}
                        </li>
                    </draggable>
                    <draggable element="ul"
                               v-model="dragdatato"
                               :options="{group:'draggableelements'}"
                               class="draggable"
                               @update="updateDraggable"
                               @add="updateDraggable"
                               @remove="updateDraggable"
                    >
                        <li class="draggable-element" v-for="element in dragdatato" :key="element.id">
                            {{ element.name }}
                        </li>
                    </draggable>
                </div>
            </transition>

        </div>


        <table>
            <tbody>
            <td v-if="position>0" @click="back()" class="submit back">&lt;</td>
            <td v-else class="submit">&nbsp;</td>
            <td v-if="position<questionnaire.length-1" @click="forward()" class="submit forward"
                :class="{'has-answer':hasAnswer, 'not-answered':!hasAnswer}">&gt;
            </td>
            <td v-else class="submit">&nbsp;</td>
            </tbody>
        </table>


        <div class="debug">
            <h3>Visual Debug</h3>
            Daten für den Fragebogen: <code>{{ responses }}</code>
            <h3>TODOs</h3>
            <ul>
                <li>Integration in SSI (analog SSI-Freeformat-Demo?)</li>
                <li>Parser, der die Fragenstruktur aus dem geenrierten SSI-Fragebogen extrahiert.</li>
                <li>Falls SinglePageApp:
                    <ul>
                        <li>Befragung fortsetzen ermöglichen</li>
                        <li>Reload (==alles auf Null) verhindern</li>
                    </ul>
                </li>
            </ul>

        </div>
    </div>
</template>

<script>
    import draggable from 'vuedraggable'

    export default {
        name: 'app',
        components: {
            draggable,
        },
        data() {
            return {
                responses: {},
                position: 0,
                dragdata: [],
                dragdatato: [],
                questionnaire: [
                    {
                        'name': 'intro',
                        'type': 'text',
                        'ask': 'Willkommen bei der Fragebogen-Demo!',
                        'tip': 'Bitte beachten: Dies ist nur eine (fast) funktionslose Demo!',
                    },
                    {
                        'name': 'drag1',
                        'type': 'draganddrop',
                        'ask': 'Bitte wählen Sie die von Ihnen bevorzugten Namen und sortieren sie nach Präferenz',
                        'tip': 'Ziehen sie dazu die Namen aus der linken Box in die rechte und sortieren Sie sie dort nach Ihren Präferenzen.',
                        'dragdata': [
                            {id: 1, name: 'Andreas'},
                            {id: 2, name: 'Sepp'},
                            {id: 3, name: 'Carol'},
                            {id: 4, name: 'NoName'},
                        ],
                        'dragdatato': [],
                    },
                    {
                        'name': 'drag2',
                        'type': 'draganddrop',
                        'ask': 'Bitte wählen Sie die von Ihnen bevorzugten Farben',
                        'tip': 'Wie eben bei den Namen ...',
                        'dragdata': [
                            {id: 1, name: 'Rot'},
                            {id: 2, name: 'Grün'},
                            {id: 3, name: 'Blau'},
                            {id: 4, name: 'gelb'},
                        ],
                        'dragdatato': [],
                    },
                    {
                        'name': 'zuf',
                        'type': 'singleselect',
                        'ask': 'Wie zufrieden sind Sie mit diesem Fragebogen?',
                        'responses': [
                            {'code': 1, 'label': 'vollkommen zufrieden'},
                            {'code': 2, 'label': 'sehr zufrieden'},
                            {'code': 3, 'label': 'zufrieden'},
                            {'code': 4, 'label': 'weniger zufrieden'},
                            {'code': 5, 'label': 'unzufrieden'},
                            {'code': 99, 'label': 'weiß nicht/keine Angabe', 'dk': true},
                        ],
                    },
                    {
                        'name': 'outro',
                        'type': 'text',
                        'ask': 'Danke! Das war\'s.',
                    },
                ],
            }
        },
        computed: {
            currentquestion: function () {
                if (this.position < 0 || this.position > this.questionnaire.length - 1) {
                    return {}
                } else {
                    return this.questionnaire[this.position]
                }
            },
            hasAnswer() {
                switch (this.currentquestiontype) {
                    case 'text':
                        return true
                        break
                    case 'singleselect':
                        return this.responses[this.currentquestionname]
                        break;
                    case 'draganddrop':
                        return this.responses[this.currentquestionname] && this.responses[this.currentquestionname].length > 0
                        break;
                    default:
                        return true
                }
                return this.responses[this.currentquestionname]
            },
        },
        methods: {
            setresponse(name, value) {
                this.$set(this.responses, name, value)
            },
            forward() {
                this.position++
                this.populatedragdata()
            },
            back() {
                this.position--;
                this.populatedragdata()
            },
            updateDraggable() {
                this.currentquestion.dragdata = this.dragdata
                this.currentquestion.dragdatato = this.dragdatato
                this.$set(this.responses, this.currentquestion.name, this.dragdatato.map(function (el) {
                    return el.id
                }))
            },
            populatedragdata() {
                if (this.questionnaire[this.position].type==='draganddrop') {
                    this.dragdata = this.questionnaire[this.position].dragdata
                    this.dragdatato = this.questionnaire[this.position].dragdatato
                }
            }
        },
    }
</script>

<style>
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        padding: 3em;
    }

    ul.draggable {
        width: 100px;
        margin: 5px;
        display: block;
        min-height: 170px;
        display: inline-block;
        vertical-align: top;
        border: 1px solid #666;
        padding: 5px;
    }

    li.draggable-element {
        border: 1px solid #666;
        margin: 5px;
        border-radius: 5px;
        padding: 3px;
        display: block;
        list-style-type: none;
    }

    .sortable-chosen {
        background-color: #ccc;
    }

    .sortable-ghost {
        opacity: 0.4;
        background-color: #ccc;
    }

    td.submit {
        font-size: 2em;
        font-weight: bold;
        color: #666;
        display: inline-block;
        width: 2em;
    }

    td.submit.not-answered {
        color: red;
    }

    .debug {
        margin-top: 80px;
        padding: 5px;
        background-color: #42b983;
        color: white;
    }

    .question-name {
        font-family: monospace;
        color: orangered;
    }

    .question-name::before {
        content: '[';
    }

    .question-name::after {
        content: ']';
    }

    .question-ask {
        font-weight: bold;
    }

    .question-tip {
        display: block;
    }

    ul.singleselect {
        padding: 0px;
        padding-left: 10px;
    }

    ul.singleselect li {
        list-style-type: none;
    }

    li.dk {
        color: #666;
        font-style: italic;
        margin-top: .5em;
    }

    label.selected {
        font-weight: bold;
    }

    /** Transitions */

    .drag-wrapper {
        display: inline-block;
    }

    ul.singleselect {
        position: relative;
    }

    .disco-enter-active {
        animation-name: my-disco;
        animation-duration: 1000ms;
    }

    .disco2-enter-active {
        animation-name: my-disco2;
        animation-duration: 1000ms;
    }

    .disco-leave-active,
    .disco2-leave-active {
    }

    @keyframes my-disco {
        0% {
            transform: scale(0);
            opacity: 0;
            background-color: inherit;
        }
        50% {
            transform: scale(1.1);
            opacity: 0.5;
            /*background-color: yellow;*/
        }
        100% {
            transform: scale(1);
            opacity: 1;
            background-color: inherit;
        }
    }

    @keyframes my-disco2 {
        0% {
            opacity: 0;
            color: transparent;
            top: -200px;
        }
        100% {
            opacity: 1;
            color: inherit;
            top: 0;
        }
    }
</style>
