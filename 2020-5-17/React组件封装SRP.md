# React组件封装SRP-实践

> 文章来自于
>
>  [英文原版](https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component/)
>
> [掘金翻译版](https://juejin.im/post/5d4acb28e51d45620771f082)

#### 职责单一

我用一个查询学生列表举例,应该将这个功能分成两个组件`<StudentListFetch>`和`<StudentListInfo>`.

`<StudentListFetch>`组件用途是进行`http`请求.`<StudentListInfo>`组件的用途是将`studentList`数组进行展示.

#### 想到的优点

1. **请求/展示**逻辑修改的时候,只会修改一个组件,不会过多的涉及更多的逻辑.
2. **请求**组件的重用
3. 性能的优化`memo`

#### 开始

```javascript
// StudentListFetch
import React, { useCallBack } from 'react';

import axios from 'axios';

import StudentListInfo from './components/StudentListInfo';

export default ({ classUuid }) => {
    const {studentList, setStudentList} = useState([])
    
    const queryStudentList = useCallBack(async (classUuid) => {
        let { data: {
            studentList
        } } = await axios.get('/queryStudentList', {
            param: {
                classUuid
            }
        })
        
        setStudentList(studentList);
    }, []);
    
    useEffect(() => {
        queryStudentList(classUuid);
    }, [classUuid, queryStudentList])
    
    return <StudentListInfo studentList={studentList} />;
}
```

```javascript
// StudentListInfo
import React from 'react';

export default ({ studentList }) => {
    return (<>
    	{ studentList.map(student => <span key={student.uuid}>{student.name}</span>) }
    </>);
}
```

