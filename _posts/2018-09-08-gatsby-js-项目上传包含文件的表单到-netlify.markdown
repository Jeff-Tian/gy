---
title: gatsby js 项目上传包含文件的表单到 netlify
layout: post
date: '2018-09-08 11:41:06 +0800'
categories: 程序员日常 
---

### 背景
一个全静态的基于 `gatsby js` 创建的网站，托管在 `netlify` 平台上，如何上传包含文件的表单呢？
先参考了一篇[官方博客文章]（https://www.netlify.com/blog/2017/07/20/how-to-integrate-netlifys-form-handling-in-a-react-app/），工作得很好，但是博客中的代码示例并没有展示文件上传功能。

### 官方博客中可以工作的示例代码

```javascript
<script type="text/babel">

  const encode = (data) => {
    return Object.keys(data)
        .map(key => encodeURIComponent(key) + "=" + encodeURIComponent(data[key]))
        .join("&");
  }

  class ContactForm extends React.Component {
    constructor(props) {
      super(props);
      this.state = { name: "", email: "", message: "" };
    }

    /* Here’s the juicy bit for posting the form submission */

    handleSubmit = e => {
      fetch("/", {
        method: "POST",
        headers: { "Content-Type": "application/x-www-form-urlencoded" },
        body: encode({ "form-name": "contact", ...this.state })
      })
        .then(() => alert("Success!"))
        .catch(error => alert(error));

      e.preventDefault();
    };

    handleChange = e => this.setState({ [e.target.name]: e.target.value });

    render() {
      const { name, email, message } = this.state;
      return (
        <form onSubmit={this.handleSubmit}>
          <p>
            <label>
              Your Name: <input type="text" name="name" value={name} onChange={this.handleChange} />
            </label>
          </p>
          <p>
            <label>
              Your Email: <input type="email" name="email" value={email} onChange={this.handleChange} />
            </label>
          </p>
          <p>
            <label>
              Message: <textarea name="message" value={message} onChange={this.handleChange} />
            </label>
          </p>
          <p>
            <button type="submit">Send</button>
          </p>
        </form>
      );
    }
  }

  ReactDOM.render(<ContactForm />, document.getElementById("root"));

</script>
```

### 一点分析：

要上传文件，可以在表单里加上 `<input type="file">`，相应地对 `handleChange` 方法要改成如果是 `files`，就要设置 `e.target.files` 而非 `e.target.value`，因为对于文件选择框，其 `value` 只是所选文件的第一个文件名。

另外官博的示例代码中有一个 `encode` 方法，在上传文本内容时没有问题，但要是上传文件，它就帮不上忙了。看来必须得对这个 `encode` 方法做改造。自然想到要用 `FormData` 来搞定，然后要注意的是去掉那个 `urlencoded` 的 `header` 指定。

### 解决方案：

改造后的 `encode` 方法如下：

```javascript
const encode = (data) => {
  const formData = new FormData()
  Object.keys(data)
    .map(key => {
      if (key === 'files') {
        for (const file of data[key]) {
          formData.append(key, file, file.name)
        }
      } else {
        formData.append(key, data[key])
      }
    })
  return formData
}
```

完整的示例代码如下：
```javascript
<script type="text/babel">

    const encode = (data) => {
    const formData = new FormData()
    Object.keys(data)
        .map(key => {
        if (key === 'files') {
            for (const file of data[key]) {
            formData.append(key, file, file.name)
            }
        } else {
            formData.append(key, data[key])
        }
        })
    return formData
    }

  class ContactForm extends React.Component {
    constructor(props) {
      super(props);
      this.state = { name: "", email: "", message: "" };
    }

    /* Here’s the juicy bit for posting the form submission */

    handleSubmit = e => {
      fetch("/", {
        method: "POST",
        body: encode({ "form-name": "contact", ...this.state })
      })
        .then(() => alert("Success!"))
        .catch(error => alert(error));

      e.preventDefault();
    };

    handleChange = (e) => {
        if(name !== 'files'){
            this.setState({ [e.target.name]: e.target.value });
        } else {
            this.setState({
                [name]: e.target.files
            })
        }
    }

    render() {
      const { name, email, message } = this.state;
      return (
        <form onSubmit={this.handleSubmit}>
          <p>
            <label>
              Your Name: <input type="text" name="name" value={name} onChange={this.handleChange} />
            </label>
          </p>
          <p>
            <label>
              Your Email: <input type="email" name="email" value={email} onChange={this.handleChange} />
            </label>
          </p>
          <p>
            <label>
              Message: <textarea name="message" value={message} onChange={this.handleChange} />
            </label>
          </p>
          <p>
            <label>
                Files: <input id="files" name="files" type="file" multiple value={files} onChange={this.handleChange} />
            </label>
          </p>
          <p>
            <button type="submit">Send</button>
          </p>
        </form>
      );
    }
  }

  ReactDOM.render(<ContactForm />, document.getElementById("root"));

</script>
```

### 项目实例：

- [帕累托火焰](https://fire.pa-pa.me) 的表单上传中使用了如上解决方案。如需体验，需要先登录。