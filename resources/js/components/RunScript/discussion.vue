<template>
    <div>
        <div
            class="discussion"
            v-for="(question, key) in questionWithReplace"
            :key="key"
        >
            <div
                class="bubble sender first"
                v-html="question.text"
            ></div>

            <div
                v-if="answers[key]"
                class="bubble recipient first"
                v-html="answers[key].name"
            ></div>
        </div>

        <div
            v-if="startScript"
            class="variables"
            v-for="variable in variablesInCurrentScript"
            :key="variable.id"
        >
            <div class="input-group-sm">
                <label
                    :for="variable.code"
                >
                    {{ variable.name }}
                </label>

                <input
                    :id="variable.code"
                    type="text"
                    @input="changeVariableValue(variable.code, $event.target.value)"
                />
            </div>
        </div>
    </div>
</template>

<script>
    import {mapActions, mapGetters} from 'vuex';

    export default {
        name: "discussion",

        props: ['questions', 'answers', 'startScript', 'clickEnd'],

        data: () => ({

            /**
             * Map: код => значение переменных для скрипта
             */
            replaceMap: new Map(),

            questionWithReplace: []
        }),

        async mounted () {
            this.$store.dispatch('setCurrentScriptId', this.$route.params.id);
            await this.$store.dispatch('setVariablesInCurrentScript');

            for (const variable of this.variablesInCurrentScript) {
                this.replaceMap.set(variable.code, '');
            }
        },

        watch: {
            startScript (val) {
                this.questionWithReplace = this.getQuestionWithReplaceFirst();
            },

            questions (val) {
                if (val.length > 1) {
                    this.questionWithReplace = this.getQuestionWithReplaceFirst();
                }
            },

            clickEnd () {
                let questions = this.questionWithReplace.map(el => {
                    return el.text;
                }).join(',');

                let answers = this.answers.map(el => {
                    return el.name;
                }).join(',');

                this.$emit('isPrepareDialogResultData', {
                    questions, answers
                });
            }
        },

        computed: {
            ...mapGetters([
                'currentScriptId',
                'variablesInCurrentScript'
            ])
        },

        methods: {
            ...mapActions([
                'setVariablesInCurrentScript'
            ]),

            /**
             * Событие изменения значения переменной.
             */
            changeVariableValue (code, inputVal) {
                for (let [key, val] of this.replaceMap) {
                    if (key == code) {
                        this.replaceMap.set(code, inputVal);
                    }
                }

                this.questionWithReplace = this.getQuestionWithReplace(code);
            },

            /**
             * Начальная загрузка
             *
             * @returns {[]}
             */
            getQuestionWithReplaceFirst () {
                let replacedObject = {};

                const questionWithReplace = this.questions.map(question => {
                    replacedObject = JSON.parse(JSON.stringify(question));

                    for (let [code, val] of this.replaceMap) {
                        replacedObject.text = replacedObject.text.replace(
                            new RegExp('{{' + code + '}.*}', 'gim'),
                            '{{' + code + '}' + val + '}'
                        );
                    }

                    return JSON.parse(JSON.stringify(replacedObject));
                });

                return questionWithReplace;
            },

            /**
             * Заменяет в массиве this.questions то, что является переменной значением из input
             *
             * @returns {[]}
             */
            getQuestionWithReplace (code) {
                let replacedObject = {};

                const questionWithReplace = this.questionWithReplace.map(question => {
                    for (let [curCode, val] of this.replaceMap) {
                        if (code == curCode) {
                            replacedObject.text = question.text.replace(
                                new RegExp('{{' + code + '}.*}', 'gi'),
                                '{{' + code + '}' + val + '}'
                            );

                            return JSON.parse(JSON.stringify(replacedObject));
                        }
                    }

                    return question;
                });

                return questionWithReplace;
            }
        }
    }
</script>

<style scoped>
    .discussion {
        display: flex;
        flex-direction: column;
    }

    .discussion > .bubble {
        border-radius: 1em;
        padding: 0.25em 0.75em;
        margin: 1.0625em;
        max-width: 470px;
        min-width: 470px;
    }

    .discussion > .bubble.sender {
        align-self: flex-start;
        background-color: cornflowerblue;
        color: #fff;
    }

    .sender ol {
        text-align: left;
    }

    .discussion > .bubble.recipient {
        align-self: flex-end;
        background-color: #efefef;
    }

    .discussion > .bubble.sender.first { border-bottom-left-radius: 0.1em; }
    .discussion > .bubble.sender.last { border-top-left-radius: 0.1em; }
    .discussion > .bubble.sender.middle {
        border-bottom-left-radius: 0.1em;
        border-top-left-radius: 0.1em;
    }

    .discussion > .bubble.recipient.first { border-bottom-right-radius: 0.1em; }
    .discussion > .bubble.recipient.last { border-top-right-radius: 0.1em; }
    .discussion > .bubble.recipient.middle {
        border-bottom-right-radius: 0.1em;
        border-top-right-radius: 0.1em;
    }
</style>
