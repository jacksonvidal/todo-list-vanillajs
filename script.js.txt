'use strict'

var dragSrcElement = null;

const createTodo = (todoListElement, text) => {
    if (text !== "") {
        let li = document.createElement("li")
        let span = document.createElement("span")
        let textNode = document.createTextNode(text)
        let doneButton = document.createElement("button");

        li.draggable = true;

        addDragAndDropHandlers(li);

        doneButton.innerHTML = "Done";
        doneButton.className = "set-done"

        span.appendChild(textNode);
        li.appendChild(span);
        li.appendChild(doneButton);

        todoListElement.appendChild(li);
    }
};

const searchDone = (doneListElement, text) => {
    let items = doneListElement.getElementsByTagName("li");

    for (let i = 0; i < items.length; i++) {
        let doneItem = items[i].querySelector("span").innerText;

        if (doneItem.trim().toUpperCase().indexOf(text.trim().toUpperCase()) > -1) {
            items[i].style.display = "";
        } else {
            items[i].style.display = "none";
        }
    }
};

//create a done item in a given list of done, did that in different functions so 
//the scope of "done" an "todo" doesn't mix so they can grow into different ways
const done = (doneListElement, text) => {

    let li = document.createElement("li");
    let span = document.createElement("span");
    let textNode = document.createTextNode(text)
    let removeButton = document.createElement("button");

    li.className = "done-items showing"

    removeButton.innerHTML = "Remove";
    removeButton.className = "removeMe";

    removeButton.addEventListener('click', function (e) {
    	    	let path = e.path || (e.composedPath && e.composedPath());
    	
        let li = path.find(p => p.tagName.toLowerCase() === "li");
        removeDone(li);
    })

    span.style.textDecoration = "line-through";

    span.appendChild(textNode);

    li.appendChild(span);
    li.appendChild(removeButton);

    doneListElement.appendChild(li);
};

const removeDone = (item) => {
    item.parentElement.removeChild(item);
};


const clearDone = (doneList) => {
    doneList.innerHTML = "";
};

const handleDoneForm = (searchInput, removeButton, disable) => {
    searchInput.disabled = disable;
    removeButton.disabled = disable;
};

const addDragAndDropHandlers = (el) => {
    el.addEventListener('dragstart', function (e) {
        dragSrcElement = this;

        e.dataTransfer.effectAllowed = 'move';
        e.dataTransfer.setData('text/html', this.outerHTML);

        this.classList.add('dragElem');
    }, false);

    el.addEventListener('dragover', function (e) {

        if (e.preventDefault) {
            e.preventDefault();
        }

        this.classList.add('over');

        e.dataTransfer.dropEffect = 'move';

        return false;
    }, false);

    el.addEventListener('dragleave', function (e) {
        this.classList.remove('over');
    }, false);

    el.addEventListener('drop', function (e) {
        if (e.stopPropagation) {
            e.stopPropagation();
        }

        if (dragSrcElement != this) {
            this.parentNode.removeChild(dragSrcElement);

            var dropHTML = e.dataTransfer.getData('text/html');
            this.insertAdjacentHTML('beforebegin', dropHTML);

            var dropElem = this.previousSibling;
            addDragAndDropHandlers(dropElem);

        }

        this.classList.remove('over');
        return false;
    }, false);

    el.addEventListener('dragend', function (e) {
        this.classList.remove('over');
    }, false);
};

window.onload = () => {

    const btnAdd = document.getElementById("add");
    const todoText = document.getElementById("todo-item");
    const doneText = document.getElementById("done-item");
    const todoList = document.getElementById("todo-list");
    const doneList = document.getElementById("done-list");
    const btnRemoveAll = document.getElementById("removeAll");

    //disable search form at done list
    handleDoneForm(doneText, btnRemoveAll, true);

    //set this general click handle because of the recreation of the li elements on drag 'n drop
    document.body.addEventListener("click", (e) => {
        if (e.target && e.target.classList.contains("set-done")) {
        	let path = e.path || (e.composedPath && e.composedPath());
        	
            let li = path.find(p => p.tagName.toLowerCase() === "li")
            let todo = li.querySelector("span").innerText;

            done(doneList, todo);
            li.parentElement.removeChild(li);
            handleDoneForm(doneText, btnRemoveAll, false);
        }
    })

    btnAdd.addEventListener("click", () => {
        createTodo(todoList, todoText.value);
        todoText.value = "";
    });

    todoText.addEventListener("keypress", (e) => {
        if (e.keyCode === 13) {
            createTodo(todoList, todoText.value);
            todoText.value = "";
        }
    });

    doneText.addEventListener("keyup", () => {
        searchDone(doneList, this.value)
    });

    btnRemoveAll.addEventListener("click", () => {
        clearDone(doneList);
        handleDoneForm(doneText, btnRemoveAll, true);
    });
}
