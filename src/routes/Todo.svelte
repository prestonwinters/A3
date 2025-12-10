<script>
    import {onMount} from 'svelte';
    let todoItem = $state('');
    let todoDate = $state();
    let todoList = $state([]);
    let todayDate = $state();
    //$inspect(todayDate);

    onMount(() => {
        let storedList = localStorage.getItem("storedList");
        if (storedList) {
            todoList = (JSON.parse(storedList));
        }

        checkDueDates();
        
        let itemDueSoon =todoList.some(item => item.dueSoon === true);
        console.log(itemDueSoon);
        if(itemDueSoon){
            navigator.vibrate((200, 200, 200));
        }
    })

    function updateList() {
        return localStorage.setItem("storedList", JSON.stringify(todoList));
    }

    // add todoItem to todoList
    function addItem(event) {
        event.preventDefault();
        if (todoItem.trim().length === 0) {
            return;
        }
        let itemDueDate = Math.floor(new Date(todoDate).getTime() / 1000);
        console.log(itemDueDate);
        todoList = [...todoList, {
            text: todoItem,
            dueDate: itemDueDate,
            dueSoon: false,
            late: false,
            done: false
        }];
        checkDueDates();
        updateList();
        todoItem = '';
    }

    function checkDueDates() {
        todayDate = Math.floor(new Date().getTime() / 1000);
        
        todoList.forEach(item => {
            let daysDue = calculateDelta(todayDate, item.dueDate);
            if (daysDue < 7) {
                item.dueSoon = true;
            } 
            if (daysDue < 0) {
                item.late = true;
            }
        })
    }

    // remove todoItem from todoList
    function removeItem(index) {
        todoList.splice(index,1);
        updateList();
    }

    function calculateDelta(date1, date2) {
        return Math.round((date2 - date1) / 86400);
    }

    function clearAll() {
        todoList = '';
        localStorage.clear();
    }

    function clearDone() {
        todoList = todoList.filter(item => !item.done);
        updateList();
    }

</script>

    <form onsubmit={addItem}>
        <input type="text" bind:value={todoItem}>
        <input type="date" bind:value={todoDate}>
        <button type="submit">Add</button>
    </form>

    <div class="todo-list">
        <ul>
            {#each todoList as item, index}
                <li>
                    <input type="checkbox" bind:checked={item.done} onchange={updateList}>
                    <span class:done={item.done} class:late={item.late} class:soon={item.dueSoon}>{item.text}: 
                        {#if item.late}
                            Late!
                        {:else if item.dueSoon}
                            Due Soon!
                        {:else}
                            due in {calculateDelta(todayDate, item.dueDate)} days.
                        {/if}
                    
                    </span>
                    <button type="button" onclick={() => removeItem(index)}>&times;</button>
                </li>
            {/each}
        </ul>
    </div>


    <button onclick={clearAll}>Clear List</button>
    <button onclick={clearDone}>Clear Done</button>
<style>

    ul {
        list-style: none;
    }
    input[type="checkbox"] {
        height: 20px;
        width: 20px;
        accent-color: rgb(255, 255, 255);

    }
    li span.done, li span.soon.done {
        text-decoration: line-through;
        color: rgb(0, 255, 0);
    }
    li span.late.soon {
        color: red;
    }
    li span.soon {
        color: rgb(255, 251, 0);
    }
    
</style>