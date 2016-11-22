# wp_tree
###插件实现的功能
树的无限分类，将dom操作与逻辑分离开，理论上可适配不同的树的ui
###实现基本结构说明
>由一个树对象多个节点对象组成，节点组成一个链表，树持有链表的开始节点

    function Tree(style) {
        if (Style[style]) {
            this.style = Style[style];
            this.startPoint = "";
            this.events = { single: "", expanded: "", close: "" };
            return this;
        }
    }

    function Node(tree, level, obj, type, next, pre, parent, child) {
        this.tree = tree;
        this.obj = obj;
        this.expanded = false;
        this.level = level;
        this.next = next || "";
        this.pre = pre || "";
        this.parent = parent || "";
        this.child = child || "";
        this.type = type || 1;
        this.clickEvent = [];
        this.attribute = {};
    }


###使用说明
>不同ui的dom接口实现

        Style.style1 = {
        //这里的i是指树的层级，从第一级开始，这是考虑到有可能不同级的节点dom不一致
        getSingleNodeHtml: function(i) { //没有子节点的节点html
            return "<div class='tree-item' style='display: block;'> " + "<i class = 'tree-dot' > </i>" + "<div class = 'tree-item-name' > Modern Elements </div>" + "</div>";
        },
        getMulNodeHtml: function(i) { //有子节点的节点html
            return "   <div class='tree-folder' style='display: block;'>" + "<div class='tree-folder-header'>" + "<i class='fa fa-folder'></i>" + "<div class='tree-folder-name'>View Category" + "<div class='tree-actions'></div>" + "</div>" + "</div>" + "<div class='tree-folder-content' style='display: none;'>" + "</div>" + "<div class='tree-loader' style='display: none;'><img src='images/input-spinner.gif'></div>" + "</div> ";
        },
        setSingleText: function(i, obj, text) { //设置节点的text方法
            obj.children().eq(1).text(text);
        },
        getSingleText: function(i, obj) { //获取节点text方法
            return obj.children().eq(1).text();
        },
        setMulText: function(i, obj, text) {
            obj.children().eq(0).children().eq(1).text(text);
        },
        getMulText: function(i, obj) {
            return obj.children().eq(0).children().eq(1).text();
        },
        expand: function(i, obj) { //展开节点的dom操作
            obj.children().eq(0).children().eq(0).removeClass("fa-folder").addClass("fa-folder-open");
            obj.children().eq(1).css("display", "block");
        },
        close: function(i, obj) { //收缩节点的dom操作
            obj.children().eq(0).children().eq(0).removeClass("fa-folder-open").addClass("fa-folder");
            obj.children().eq(1).css("display", "none");
        },
        addChild: function(i, parent, child) { //向一个父节点添加第一个子节点的dom操作
            parent.children().eq(1).append(child);
        }，
        before: function(i, newO, o) {//节点新节点插入树摸个节点之前的dom操作
            o.before(newO);
        },
        after: function(i, o, newO) {//节点新节点插入树摸个节点之后的dom操作
            o.after(newO);
        }
    };
>初始化数据结构

    var data = [{
        text: "tree0",
        child: [{
            text: "tree00",
            child: [{
                text: "tree001"
            }, {
                text: "tree002"
            }]
        }, {
            text: "tree01",
            attribute: {//自定义业务属性
                id: "11111"
            }
        }, {
            text: "tree02"
        }]
    }, {
        text: "tree1",
        child: [{
            text: "tree10"
        }, {
            text: "tree11"
        }, {
            text: "tree12"
        }]
    }, {
        text: "tree2",
        child: [{
            text: "tree20"
        }, {
            text: "tree21"
        }, {
            text: "tree22"
        }]
    }];
    
>业务事件定义

    var event = {
        //这三个函数的this都指向node对象，可以根据node.level和node.type对不同等级的树以及展开和收缩时实现不同事件触发
        single: function() {//没有子节点的click事件
            console.log("single:" + this.getText());
        },
        expanded: function() {//节点展开时触发事件
            console.log("expanded:" + this.getText());
        },
        close: function() {//节点收缩触发事件
            console.log("close:" + this.getText());
        }
    }; 
    
>实例化树的对象并渲染dom,因为每个tree都是闭包实现，所以同一页面可多个不冲突

    //这是一次性加载所有数据
    var tree1 = new WpTree("style1").setEvent(event).initChainData(data).inner($("#tree1"));
    
    //逐级点击加载数据
    var tree1 = new WpTree("style1").setEvent(event).setUrl(//设置每一级请求的url
            function(node) {
                if (node) {
                    var bookId = "",
                        catalogId = "";
                    if (node.parent) {
                        bookId = node.attribute.bookId;
                        catalogId = node.attribute.id;
                    } else {
                        bookId = node.attribute.id;
                    }
                    return "./teacherPersonal/chapterCatalog?booksId=" + bookId + "&catalogId=" + catalogId;
                } else {
                    return "./teacherPersonal/booksCatalog?subjectCode=" + subjectCode + "&gradeCode=" + gradeCode;
                }
            }).formatData(function(data, node) {//数据格式转换
            if (node) {
                return changeData2(data);
            } else {
                return changeData1(data);
            }
        }).afterLoad(function(node) {//点击张开加载动画，数据加载完成后关闭动画
            node && node.obj.children().eq(3).addClass("hidden");
        }).autoLoad($("#books"));

    
###树的结构说明
见上传的图片

###函数说明
* tree.addNode(coordinate, text, attribute) 增加节点 节点位置，节点值，节点属性
* tree.getNode(coordinate) 获取指定位置的节点
* tree.deleteNode(coordinate) 删除指定位置的节点
* node.remove 删除该节点
* node.getAttribute(key) 获取该节点指定key对应的属性值
* node.setAttribute(key,value) 设置该节点的属性



    
  
