<template>
    <main class="wrap" :class="!userSettings.theme ? 'dark__theme' : 'light__theme'">
        <div class="todo_wrapper">
            <header>
                <h1 class="todo_title"><a href="/">todo</a></h1>
                <div class="header_btns">
                    <button class="saving_switcher"
                            type="button"
                            @click="changeSavingSettings"
                            :class="{'saved': userSettings.progressSaved}"
                            :title="!userSettings.progressSaved ? 'Save my progress':
                            'Stop tracking and delete my progress'"></button>
                    <button class="theme_switcher"
                            type="button"
                            @click="userSettings.theme = !userSettings.theme"></button>
                </div>
            </header>
            <section class="todo_item todo_new_input">
                <input class="todo_item__input"
                       type="text"
                       @keyup.enter="addTask"
                       v-model="taskToAdd"
                       placeholder="Type new task and press enter to add...">
            </section>
            <section class="todo_tasks">
                <div class="todo_item todo_task"
                     :class="{ task_done: task.isDone }"
                     v-for="task in filtered"
                     :key="task.id">
                    <button class="todo_item__status"
                            type="button"
                            @click="setTaskDone(task.id, $event)">
                    </button>
                    <input class="todo_item__input"
                           type="text"
                           disabled
                           v-bind:placeholder="task.text">
                    <button class="todo_item__delete"
                            type="button"
                            @click="deleteTask(task.id)"></button>
                </div>
                <div class="todo_item todo_footer">
                    <span class="todo_item__info">{{toDoList.filter(i=>!i.isDone).length}} items left</span>
                    <div class="todo_item__filter">
                        <button class="todo_item__filter__btn active"
                                @click="filterTasks('all',$event)">All</button>
                        <button class="todo_item__filter__btn"
                                @click="filterTasks('active',$event)">Active</button>
                        <button class="todo_item__filter__btn"
                                @click="filterTasks('completed',$event)">Completed
                        </button>
                    </div>
                    <button class="btn_clear"
                            @click="deleteCompletedTasks">Clear Completed</button>
                </div>
            </section>
        </div>
    </main>
</template>

<script>
export default {
    name: 'App',
    data: function (){
        return {
            userSettings: {
                theme: false,
                progressSaved: false
            },
            theme: false,
            taskToAdd: '',
            toDoList: [],
            filter: 'all',
            filtered: []
        }
    },
    beforeMount() {
        if (localStorage.getItem('MyLittleTodo')){
            const saveData = JSON.parse(localStorage.getItem('MyLittleTodo'));
            Object.entries(saveData).forEach(([key,val]) =>  this[key] = val )
        }
        return this.filtered = this.toDoList;
    },
    computed:{
        generateNewTaskId(){
            return this.toDoList.length ? this.toDoList[this.toDoList.length - 1].id + 1 : 0;
        }
    },
    methods: {
        setTaskDone(id,e){
            e.target.parentElement.classList.toggle('task_done')
            let task = this.toDoList.findIndex(task => task.id === id);
            this.toDoList[task].isDone = !this.toDoList[task].isDone;
            return this.filterTasks(this.filter);
        },
        addTask(){
            if (this.taskToAdd.length){
                this.toDoList.push({
                    id: this.generateNewTaskId,
                    text: this.taskToAdd,
                    isDone: false
                });
                this.taskToAdd = '';
                return this.filterTasks(this.filter);
            }
        },
        deleteTask(id){
            let task = this.toDoList.findIndex(task => task.id === id);
            this.toDoList.splice(task, 1);
            return this.filterTasks(this.filter);
        },
        deleteCompletedTasks(){
            let toDelete = this.toDoList.filter(task=> task.isDone);
            this.toDoList = this.toDoList.filter(task => !toDelete.includes(task))
            return this.filterTasks(this.filter);
        },
        filterTasks(filter,el){
            if (el){
                document.querySelector('.todo_item__filter__btn.active').classList.remove('active');
                el.target.classList.add('active');
            }
            if (this.userSettings.progressSaved){
                this.saveProgress()
            }
            switch (filter){
                case 'all':
                    this.filter = 'all';
                    return this.filtered = this.toDoList;
                case 'active':
                    this.filter = 'active';
                    return this.filtered = this.toDoList.filter(task => !task.isDone);
                case 'completed':
                    this.filter = 'completed';
                    return this.filtered = this.toDoList.filter(task => task.isDone);
            }
        },
        changeSavingSettings(){
            if (!this.userSettings.progressSaved){
                this.userSettings.progressSaved = true;
                this.saveProgress();
            }
            else {
                this.userSettings.progressSaved = false;
                localStorage.removeItem('MyLittleTodo');
            }
        },
        saveProgress(){
            try {
                const userSaves = JSON.stringify({
                    toDoList: this.toDoList,
                    userSettings: this.userSettings
                });
                localStorage.setItem('MyLittleTodo', userSaves);
            }
            catch(err) {
                console.error(`Error: ${err}`)
            }
        }
    }
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Josefin+Sans:wght@400;700&display=swap');
@import './css/normalize.css';

$activeColor: hsl(220, 98%, 61%);
$defaultTransition: .3s ease-out;

@mixin color-dark {
    color: #4c4b59;
}

@mixin color-white {
    color: #fff;
}

@mixin box-shadow {
    box-shadow: 0 0 1em 0 rgba(18, 21, 30, .5);
}

@mixin fontFix {
    transform: translateY(11%); //font vertical alignment fix
}

#app {
    font-family: 'Josefin Sans', sans-serif;
    width: 100%;
    max-width: 1440px;
    min-width: 375px;
    font-size: 18px;
    margin: 0 auto;
    position: relative;
    box-sizing: border-box;
    @media all and (min-width: 375px) and (max-width: 600px) {
        font-size: 3.2vw;
    }
    .wrap {
        height: 100vh;
        overflow: auto;
        padding: 3em 0;
        &.dark__theme {
            background: hsl(235, 21%, 11%) url("assets/bg-desktop-dark.jpg") no-repeat center top;
            background-size: 1440px;
            @media all and (min-width: 375px) and (max-width: 600px) {
                background: hsl(235, 21%, 11%) url("assets/bg-mobile-dark.jpg") no-repeat center top;
                background-size: 100%;
            }
            header {
                .theme_switcher {
                    background: url("assets/icon-sun.svg") no-repeat center center;
                    background-size: contain;
                }
            }
            .todo_wrapper {
                .todo_item {
                    background: #25273c;
                    @include color-white;
                    &.todo_task {
                        border-color: #37394e;
                    }
                }
                .todo_footer {
                    .todo_item__info {
                        opacity: .3;
                    }
                    button {
                        @include color-white;
                        opacity: .3;
                    }
                    @media all and (min-width: 375px) and (max-width: 600px) {
                        .todo_item__filter {
                            background: #25273c;
                            @include color-white;
                        }
                    }
                }
            }
        }
        &.light__theme {
            background: #fafafa url("assets/bg-desktop-light.jpg") no-repeat center top;
            background-size: 1440px;
            @media all and (min-width: 375px) and (max-width: 600px) {
                background: hsl(235, 21%, 11%) url("assets/bg-mobile-light.jpg") no-repeat center top;
                background-size: 100%;
            }
            header {
                .theme_switcher {
                    background: url("assets/icon-moon.svg") no-repeat center center;
                    background-size: contain;
                }
            }
            .todo_wrapper {
                .todo_item {
                    background: #ffffff;
                    @include color-dark;
                    &.todo_task {
                        border-color: #e6e5ea;
                    }
                    &:not(.task_done) {
                        .todo_item__status {
                            &:hover {
                                border: solid .15em transparent;
                                background: linear-gradient(to right, hsl(192, 100%, 67%), hsl(280, 87%, 65%));
                                background-origin: border-box;
                                box-shadow: .2em 10em .1em #fff inset;
                                transition: border-color $defaultTransition/2;
                            }
                        }
                    }
                }
                .todo_footer {
                    .todo_item__info {
                        opacity: .8;
                    }
                    button {
                        @include color-dark;
                        opacity: .5;
                    }
                    @media all and (min-width: 375px) and (max-width: 600px) {
                        .todo_item__filter {
                            background: #fff;
                            @include color-dark;
                        }
                    }
                }
            }
        }
        .todo_wrapper {
            width: 100%;
            max-width: 540px;
            margin: 0 auto;
            @media all and (min-width: 375px) and (max-width: 600px) {
                width: 85%;
            }
            header {
                margin-bottom: 3em;
                display: flex;
                align-items: center;
                justify-content: space-between;
                .todo_title {
                    text-transform: uppercase;
                    font-size: 3em;
                    letter-spacing: .4em;
                    @include fontFix;
                    a{
                        @include color-white;
                        text-decoration: none;
                    }
                }
                .header_btns {
                    .saving_switcher {
                        border: 0;
                        width: 2.4em;
                        height: 2.4em;
                        background: url("assets/icon-save.svg") no-repeat center center;
                        background-size: contain;
                        filter: invert(1);
                        &.saved {
                            background: url("assets/icon-delete.svg") no-repeat center center;
                            background-size: contain;
                        }
                    }
                    .theme_switcher {
                        border: 0;
                        width: 2.4em;
                        height: 2.4em;
                        margin-left: 1em;
                    }
                }
            }
            .todo_item {
                width: 100%;
                border: 0;
                padding: 1.105em;
                font-size: 1em;
                @media all and (min-width: 375px) and (max-width: 600px) {
                    padding: 1.4em;
                }
                &.todo_task {
                    display: grid;
                    grid-template-columns: 1.35em 1fr 1.35em;
                    border-bottom: .03em solid transparent;
                }
                &.todo_new_input {
                    @include box-shadow;
                    border-radius: .3em;
                    margin-bottom: 1em;
                    .todo_item__input {
                        width: 100%;
                        margin: 0 auto;
                        color: inherit;
                        &::placeholder {
                            opacity: .3;
                        }
                    }
                }
                &.task_done {
                    .todo_item__status {
                        background: url("assets/icon-check.svg") no-repeat center center,
                        linear-gradient(to right, hsl(192, 100%, 67%), hsl(280, 87%, 65%));
                        background-size: 50%, cover;
                        background-origin: border-box;
                        border: solid .15em transparent;
                    }
                    .todo_item__input {
                        text-decoration: line-through;
                        opacity: .5;
                    }
                }
                &:not(.task_done) {
                    .todo_item__status {
                        &:hover {
                            border: solid .15em transparent;
                            background: linear-gradient(to right, hsl(192, 100%, 67%), hsl(280, 87%, 65%));
                            background-origin: border-box;
                            box-shadow: .2em 10em .1em #25273c inset;
                            transition: border-color $defaultTransition/2;
                        }
                    }
                }
                &:hover {
                    .todo_item__delete {
                        visibility: visible;
                        opacity: 1;
                    }
                }
                .todo_item__status {
                    width: 1.35em;
                    height: 1.35em;
                    border-radius: 50%;
                    position: relative;
                    z-index: 1;
                    border: solid .15em #2f3146;
                }
                .todo_item__input {
                    margin: 0 1.105em;
                    @include fontFix;
                    &::placeholder {
                        opacity: 1;
                    }
                }
                .todo_item__delete {
                    visibility: hidden;
                    opacity: 0;
                    transition: opacity $defaultTransition;
                    width: 1.35em;
                    height: 1.35em;
                    background: url("assets/icon-cross.svg") no-repeat center center;
                    background-size: contain;
                    @media all and (min-width: 375px) and (max-width: 600px) {
                        visibility: visible;
                        opacity: 1;
                    }
                }
            }
            .todo_tasks {
                @include box-shadow;
                position: relative;
                .todo_item:first-child {
                    border-radius: .3em .3em 0 0;
                }
                .todo_item:last-child {
                    border-radius: 0 0 .3em .3em;
                }
                .todo_item:only-child{
                    border-radius: .3em;
                }
                .todo_footer {
                    &.todo_item {
                        display: flex;
                        align-items: center;
                        justify-content: space-between;
                        font-size: .8em;
                        .todo_item__filter {
                            @media all and (min-width: 375px) and (max-width: 600px) {
                                position: absolute;
                                bottom: -5.5em;
                                left: 0;
                                width: 100%;
                                display: flex;
                                justify-content: center;
                                padding: 1.4em;
                                border-radius: .3em;
                                @include box-shadow;
                                .todo_item__filter__btn {
                                    margin: 0 1em;
                                    font-size: 1.2em;
                                }
                            }
                            .todo_item__filter__btn {
                                font-weight: bold;
                                &.active {
                                    opacity: 1;
                                    color: $activeColor;
                                }
                                &:hover {
                                    opacity: 1;
                                    transition: opacity $defaultTransition;
                                }
                                &:not(:last-of-type){
                                    margin-right: 1.5em;
                                }
                            }
                        }
                        .btn_clear {
                            &:hover {
                                opacity: .5;
                            }
                        }
                    }
                }
            }
        }
    }
}
</style>
