<template>
    <main class="wrap dark__theme">
        <div class="todo_wrapper">
            <header>
                <h1 class="todo_title">todo</h1>
                <button class="theme_switcher" type="button"></button>
            </header>
            <section class="todo_add_task">
                <div class="todo_item todo_task todo_task_input">
<!--                    <button type="button" class="todo_item__status"></button>-->
                    <input type="text" class="todo_item__input" @keyup.enter="addTask" v-model="taskToAdd"
                           placeholder="Type new task and press enter to add...">
                </div>
            </section>
            <section class="todo_tasks">
                <div class="todo_item todo_task" :class="{ task_done: task.isDone }" v-for="task in filtered"
                     :key="task.id">
                    <button type="button" class="todo_item__status" @click="setTaskDone(task.id, $event)"></button>
                    <input type="text" class="todo_item__input" disabled v-bind:placeholder="task.text">
                    <button type="button" class="todo_item__delete" @click="deleteTask(task.id)"></button>
                </div>
            </section>
            <section class="todo_footer">
                <div class="todo_item">
                    <span class="todo_item__info">{{toDoList.filter(i=>!i.isDone).length}} items left</span>
                    <div class="todo_item__actions">
                        <button class="todo_item__actions__btn active" @click="filterTasks('all',$event)">All</button>
                        <button class="todo_item__actions__btn" @click="filterTasks('active',$event)">Active</button>
                        <button class="todo_item__actions__btn" @click="filterTasks('completed',$event)">Completed
                        </button>
                        <button class="todo_item__actions__btn" @click="deleteCompletedTasks">Clear Completed</button>
                    </div>
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
            taskToAdd: '',
            toDoList: [],
            filter: 'all',
            filtered: []
        }
    },
    beforeMount() {
        return this.filtered = this.toDoList;
    },
    computed:{
        generateNewTaskId(){
            return this.toDoList.length ? this.toDoList[this.toDoList.length - 1].id + 1 : 0;
        }
    },
    methods: {
        filterTasks(filter,el){
            if (el){
                document.querySelector('.todo_item__actions__btn.active').classList.remove('active');
                el.target.classList.add('active');
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
        setTaskDone(id,e){
            e.target.parentElement.classList.toggle('task_done')
            let task = this.toDoList.findIndex(task => task.id === id);
            this.toDoList[task].isDone = !this.toDoList[task].isDone;
            return this.filterTasks(this.filter);
        },
        addTask(){
            this.toDoList.push({
                id: this.generateNewTaskId,
                text: this.taskToAdd,
                isDone: false
            });
            this.taskToAdd = '';
            return this.filterTasks(this.filter);
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
        }
    }
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Josefin+Sans:wght@400;700&display=swap');
@import './css/normalize.css';

$primaryColor: hsl(220, 98%, 61%);
$defaultTransition: .3s ease-out;
$fontFix: translateY(11%); //font vertical alignment fix

#app {
    font-family: 'Josefin Sans', sans-serif;
    width: 100%;
    max-width: 1440px;
    min-width: 375px;
    font-size: 18px;
    margin: 0 auto;
    position: relative;
    box-sizing: border-box;
    .wrap {
        height: 100vh;
        overflow: auto;
        padding-top: 3em;
        &.dark__theme {
            background: hsl(235, 21%, 11%) url("assets/bg-desktop-dark.jpg") no-repeat center top;
            background-size: 1440px;
        }
    }
    .todo_wrapper {
        width: 100%;
        max-width: 540px;
        margin: 0 auto;
        header {
            margin-bottom: 3em;
            display: flex;
            align-items: center;
            justify-content: space-between;
            .todo_title {
                color: #fff;
                text-transform: uppercase;
                font-size: 3em;
                letter-spacing: .4em;
                transform: $fontFix;
            }
            .theme_switcher {
                border: 0;
                width: 2.4em;
                height: 2.4em;
                background: url("assets/icon-sun.svg") no-repeat center center;
                background-size: contain;
            }
        }
        .todo_item {
            width: 100%;
            background: #25273c;
            border: 0;
            padding: 1.105em;
            font-size: 1em;
            color: #fff;
            &.todo_task {
                display: grid;
                grid-template-columns: 1.35em 1fr 1.35em;
                border-bottom: .1em solid #37394e;
            }
            &.todo_task_input {
                box-shadow: 0 0 1em 0 rgba(18, 21, 30, .5);
                border-radius: .3em;
                margin-bottom: 1.5em;
                border-bottom: none;
                display: block;
                .todo_item__input {
                    width: 100%;
                    margin: 0 auto;
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
                    opacity: .2;
                }
            }
            &:not(.todo_task_input):not(:first-of-type){
                border-top: .1em solid #37394e;
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
                color: #fff;
                margin: 0 1.105em;
                transform: $fontFix;
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
            }
        }
        .todo_tasks {
            box-shadow: 0 0 1em 0 rgba(18, 21, 30, .5);
            .todo_item:first-child {
                border-radius: .3em .3em 0 0;
            }
        }
        .todo_footer {
            .todo_item {
                border-radius: 0 0 .3em .3em;
                display: flex;
                align-items: center;
                justify-content: space-between;
                font-size: 14px;
                .todo_item__info {
                    opacity: .3;
                }
                .todo_item__actions__btn {
                    color: #fff;
                    opacity: .3;
                    &.active {
                        opacity: 1;
                        color: $primaryColor;
                    }
                    &:hover {
                        opacity: 1;
                        transition: opacity $defaultTransition;
                    }
                    &:not(:last-of-type){
                        margin-right: 1.5em;
                    }
                    &:last-of-type {
                        margin-left: 2.5em;
                        &:hover {
                            opacity: .5;
                        }
                    }
                }
            }
        }
    }
}
</style>
