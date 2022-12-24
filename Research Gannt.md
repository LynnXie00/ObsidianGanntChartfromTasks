
```dataviewjs
function textParser(taskText){//input text,return object
 
    let d = taskText.indexOf("📅");
    let DueText="";
    if(d>=0){
        let i=taskText.slice(d).search(/([12]\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01]))/);
        DueText=taskText.substr(d+i,10);
    } 

    let sch = taskText.indexOf("⏳");
    let scheduledText="";
    if(sch>0){
        let i=taskText.slice(sch).search(/([12]\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01]))/);
        scheduledText=taskText.substr(sch+i,10);
    } 

    let st = taskText.indexOf("🛫");
    let startText="";
    if(st>0){
        let i=taskText.slice(st).search(/([12]\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01]))/);
        startText=taskText.substr(st+i,10);
    } 
    
    let h = taskText.indexOf("⏫");
    let m = taskText.indexOf("🔼");
    let l =taskText.indexOf("🔽");
    let PriorityText="";
    if(h>0){
        PriorityText="High";
    } 
    if(m>0){
        PriorityText="Medium";
    } 
    if(l>0){
        PriorityText="Low";
    } 
    const emojisIndex= [d,sch,st,h,m,l]
    const presentEmojiIndex= emojisIndex.filter(x => x > 0);

    let nameText=taskText.slice(0,Math.min(...presentEmojiIndex)).trim();

    console.log(taskText,Math.min(...presentEmojiIndex))
	return {    
		name: nameText,
		due: DueText,
        start:startText,
        scheduled:scheduledText,
        priority:PriorityText
		
	}
}



function loopGantt (pageArray){
	let querySections=``;
	let today = new Date().toISOString().slice(0, 10)
	for (let i = 0; i < pageArray.length; i++) {  
		let taskQuery=``;
		var taskArray = pageArray[i].file.tasks;
		//parse name, due, start, completion, scheduled,priority from task text to objects


		var taskObjs=[];
		for (let j=0; j< taskArray.length;j++){		
			taskObjs[j] = textParser(taskArray[j].text)

		}

        //determine the mermaid task parameters
		for (let j=0; j< taskObjs.length;j++){		
			let theTask = taskObjs[j];
			if (theTask.start != null){ //if has start date, use start date 
				if (theTask.completion != null){
					// if completed finish day is completion day, and mark as done.
					taskQuery+= theTask.name + `    : done,`+theTask.start+`,`+theTask.completion+`\n\n`;		
					
				}else if( theTask.due != null) {
					//if incomplete and has a due date, finish day is the due date, and mark as active
					taskQuery+= theTask.name  + `    : active,`+theTask.start+`,`+theTask.due +`\n\n`;
					
					
				}else {
					// if incomplete and has no due date, finish day is today, and mark as active
					
					taskQuery+= theTask.name + `    : active`+theTask.start+`,`+today+`\n\n`;				
				};
		
			} else{ //no start date, use today as start date
				if (theTask.completion != null){
					// if completed finish day is completion day, and mark as done.
					taskQuery+= theTask.name  + `    : done,`+today+`,`+theTask.completion+`\n\n`;		
					
				}else if( theTask.due != null) {
					//if incomplete and has a due date, finish day is the due date, and mark as active
					taskQuery+= theTask.name  + `    : active,`+today+`,`+theTask.completion+`\n\n`;
					
					
				}else {
					// if incomplete and has no due date, finish day is 2 days later, and mark as active and critical
				
					taskQuery+= theTask.name  + `    : crit, active`+today+`,`+`2d`+`\n\n`;				
				};				
				
			};
			
		};
		
		querySections+= `section  `+pageArray[i].file.name+`\n\n`+taskQuery;
		
	};
	return querySections
}


const Mermaid = `gantt

    title Gannt Charts
    
    dateFormat  YYYY-MM-DD  
    
    axisFormat %b\n %d
    
    `;


// you may change the path of your project folder below
dv.paragraph('```mermaid\n' + Mermaid + loopGantt(dv.pages('"00 Life/Project Folder/Research"'))+ '\n```');



```



