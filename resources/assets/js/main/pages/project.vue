<template>
    <div class="w-main project">

        <v-title>{{$L('项目')}}-{{$L('轻量级的团队在线协作')}}</v-title>

        <div class="w-nav">
            <div class="nav-row">
                <div class="w-nav-left">
                    <div class="page-nav-left">
                        <span class="hover" @click="addShow=true"><i class="ft icon">&#xE740;</i> {{$L('新建项目')}}</span>
                        <div v-if="loadIng > 0" class="page-nav-loading"><w-loading></w-loading></div>
                        <div v-else class="page-nav-refresh"><em @click="getLists(true)">{{$L('刷新')}}</em></div>
                    </div>
                </div>
                <div class="w-nav-flex"></div>
                <div class="w-nav-right m768-show">
                    <Dropdown @on-click="handleProject" trigger="click" transfer>
                        <Icon type="md-menu" size="18"/>
                        <DropdownMenu slot="list">
                            <DropdownItem name="myjoin">{{$L('参与的项目')}}</DropdownItem>
                            <DropdownItem name="myfavor">{{$L('收藏的项目')}}</DropdownItem>
                            <DropdownItem name="mycreate">{{$L('我管理的项目')}}</DropdownItem>
                        </DropdownMenu>
                    </Dropdown>
                </div>
                <div class="w-nav-right m768-hide">
                    <span class="ft hover" @click="handleProject('myjoin', null)"><i class="ft icon">&#xE75E;</i> {{$L('参与的项目')}}</span>
                    <span class="ft hover" @click="handleProject('myfavor', null)"><i class="ft icon">&#xE720;</i> {{$L('收藏的项目')}}</span>
                    <span class="ft hover" @click="handleProject('mycreate', null)"><i class="ft icon">&#xE764;</i> {{$L('我管理的项目')}}</span>
                </div>
            </div>
        </div>

        <w-content>
            <!-- 列表 -->
            <ul class="project-list">
                <li v-for="item in lists">
                    <div class="project-item">
                        <div class="project-head">
                            <div v-if="item.loadIng === true" class="project-loading">
                                <w-loading></w-loading>
                            </div>
                            <div class="project-title" @click="handleProject('open', item)">{{item.title}}</div>
                            <div class="project-setting">
                                <Dropdown class="right-info" trigger="click" @on-click="handleProject($event, item)" transfer>
                                    <Icon class="project-setting-icon" type="md-settings" size="16"/>
                                    <Dropdown-menu slot="list">
                                        <Dropdown-item name="open">{{$L('打开')}}</Dropdown-item>
                                        <Dropdown-item name="favor">{{$L('收藏')}}</Dropdown-item>
                                        <Dropdown-item v-if="item.isowner" name="rename">{{$L('重命名')}}</Dropdown-item>
                                        <Dropdown-item v-if="item.isowner" name="transfer">{{$L('移交项目')}}</Dropdown-item>
                                        <Dropdown-item v-if="item.isowner" name="delete">{{$L('删除')}}</Dropdown-item>
                                        <Dropdown-item v-else name="out">{{$L('退出')}}</Dropdown-item>
                                    </Dropdown-menu>
                                </Dropdown>
                            </div>
                        </div>
                        <div class="project-num" @click="handleProject('open', item)">
                            <div class="project-circle">
                                <i-circle
                                    :size="100"
                                    :trail-width="8"
                                    :stroke-width="8"
                                    :percent="selfProportion(item.self_complete, item.self_count)"
                                    stroke-linecap="round"
                                    stroke-color="#62C5FE">
                                    <div class="project-circle-box">
                                        <div class="project-circle-num">
                                            <em>{{item.self_complete}}</em>
                                            <span>{{item.self_count}}</span>
                                        </div>
                                        <div class="project-circle-title">{{$L('个人总计')}}</div>
                                    </div>
                                </i-circle>
                            </div>
                            <div class="project-situation">
                                <ul>
                                    <li>{{$L('项目总任务数')}}<em>{{item.complete + item.unfinished}}</em></li>
                                    <li>{{$L('项目已完成数')}}<em>{{item.complete}}</em></li>
                                    <li>{{$L('项目未完成数')}}<em>{{item.unfinished}}</em></li>
                                </ul>
                            </div>
                        </div>
                        <div class="project-bottom">
                            <div class="project-iconbtn">
                                <Icon class="project-iconbtn-icon" type="md-stats" />
                                <div class="project-iconbtn-text" @click.stop="handleProject('statistics', item)">{{$L('项目统计')}}</div>
                            </div>
                            <div class="project-iconbtn">
                                <Icon class="project-iconbtn-icon" type="md-filing"/>
                                <div class="project-iconbtn-text" @click.stop="handleProject('archived', item)">{{$L('已归档任务')}}</div>
                            </div>
                            <div class="project-iconbtn project-people" @click.stop="handleProject('member', item)">
                                <UserImg v-for="(uItem, uKey) in item.people_lists" class="userimg-icon" :key="uKey" :info="uItem" two-words show-title/>
                                <div v-if="item.people_count > 99" class="userimg-count" :title="item.people_count">99+</div>
                                <div v-else-if="item.people_count > 5" class="userimg-count">{{item.people_count}}</div>
                            </div>
                        </div>
                    </div>
                </li>
            </ul>
            <!-- 分页 -->
            <Page v-if="listTotal > 0" class="pageBox" :total="listTotal" :current="listPage" :disabled="loadIng > 0" :pageSize="listPageSize" @on-change="setPage" @on-page-size-change="setPageSize" :page-size-opts="[20,40,60,100]" placement="top" transfer show-elevator show-sizer show-total :simple="windowMax768"></Page>
        </w-content>

        <Modal
            v-model="addShow"
            :title="$L('新建项目')"
            :closable="false"
            :mask-closable="false"
            class-name="simple-modal">
            <Form ref="add" :model="formAdd" :rules="ruleAdd" :label-width="80" @submit.native.prevent>
                <FormItem prop="title" :label="$L('项目名称')">
                    <Input type="text" v-model="formAdd.title"></Input>
                </FormItem>
                <FormItem prop="labels" :label="$L('项目模板')">
                    <Select v-model="formAdd.template" @on-change="(res) => {$set(formAdd, 'labels', labelLists[res].value)}">
                        <Option v-for="(item, index) in labelLists" :value="index" :key="index">{{ item.label }}</Option>
                    </Select>
                </FormItem>
                <FormItem :label="$L('项目流程')">
                    <div style="line-height:38px">
                        <span v-for="(item, index) in formAdd.labels">
                            <span v-if="index > 0">&gt;</span>
                            <Tag @on-close="() => { formAdd.labels.splice(index, 1)}" closable size="large" color="primary">{{item}}</Tag>
                        </span>
                    </div>
                    <div v-if="formAdd.labels.length > 0" style="margin-top:4px;"></div>
                    <div style="margin-bottom:-16px">
                        <Button icon="ios-add" type="dashed" @click="addLabels">{{$L('添加流程')}}</Button>
                    </div>
                </FormItem>
            </Form>
            <div slot="footer">
                <Button type="default" @click="addShow=false">{{$L('取消')}}</Button>
                <Button type="primary" :loading="loadIng > 0" @click="onAdd">{{$L('添加')}}</Button>
            </div>
        </Modal>

        <WDrawer v-model="projectDrawerShow" maxWidth="1000">
            <Tabs v-if="projectDrawerShow" v-model="projectDrawerTab">
                <TabPane :label="$L('已归档任务')" name="archived">
                    <project-archived :canload="projectDrawerShow && projectDrawerTab == 'archived'" :projectid="handleProjectId"></project-archived>
                </TabPane>
                <TabPane :label="$L('项目统计')" name="statistics">
                    <project-statistics :canload="projectDrawerShow && projectDrawerTab == 'statistics'" :projectid="handleProjectId"></project-statistics>
                </TabPane>
                <TabPane :label="$L('成员管理')" name="member">
                    <project-users :canload="projectDrawerShow && projectDrawerTab == 'member'" :projectid="handleProjectId"></project-users>
                </TabPane>
            </Tabs>
        </WDrawer>

        <WDrawer v-model="projectListDrawerShow" maxWidth="740">
            <Tabs v-if="projectListDrawerShow" v-model="projectListDrawerTab">
                <TabPane :label="$L('参与的项目')" name="myjoin">
                    <project-my-join :canload="projectListDrawerShow && projectListDrawerTab == 'myjoin'"></project-my-join>
                </TabPane>
                <TabPane :label="$L('收藏的项目')" name="myfavor">
                    <project-my-favor :canload="projectListDrawerShow && projectListDrawerTab == 'myfavor'"></project-my-favor>
                </TabPane>
                <TabPane :label="$L('管理的项目')" name="mycreate">
                    <project-my-manage :canload="projectListDrawerShow && projectListDrawerTab == 'mycreate'"></project-my-manage>
                </TabPane>
            </Tabs>
        </WDrawer>
    </div>
</template>

<style lang="scss" scoped>
    .project {
        ul.project-list {
            padding: 5px;
            max-width: 2200px;
            margin: 0 auto;
            li {
                float: left;
                width: 20%;
                display: flex;
                @media (max-width: 2000px) {
                    width: 25%;
                }
                @media (max-width: 1400px) {
                    width: 33.33%;
                }
                @media (max-width: 1080px) {
                    width: 50%;
                }
                @media (max-width: 640px) {
                    width: 100%;
                }
                .project-item {
                    flex: 1;
                    margin: 10px;
                    width: 100%;
                    height: 313px;
                    padding: 20px;
                    background-color: #ffffff;
                    border-radius: 4px;
                    display: flex;
                    flex-direction: column;
                    .project-head{
                        display: flex;
                        flex-direction: row;
                        .project-loading {
                            width: 18px;
                            height: 18px;
                            margin-right: 6px;
                            margin-top: 3px;
                        }
                        .project-title{
                            flex: 1;
                            font-size: 16px;
                            padding-right: 6px;
                            overflow:hidden;
                            text-overflow:ellipsis;
                            white-space:nowrap;
                            color: #333333;
                            cursor: pointer;
                        }
                        .project-setting{
                            width: 30px;
                            text-align: right;
                            .project-setting-icon {
                                cursor: pointer;
                                color: #333333;
                                &:hover {
                                    color: #0396f2;
                                }
                            }
                        }
                    }
                    .project-num {
                        flex: 1;
                        padding: 34px 0;
                        display: flex;
                        flex-direction: row;
                        align-items: center;
                        justify-content: center;
                        cursor: pointer;
                        position: relative;
                        &:before {
                            content: "";
                            position: absolute;
                            width: 1px;
                            height: 60%;
                            background-color: #EFEFEF;
                        }
                        .project-circle {
                            flex: 1;
                            text-align: center;
                            margin-right: 10px;
                            .project-circle-box {
                                display: flex;
                                flex-direction: column;
                                align-items: center;
                                justify-content: center;
                                .project-circle-num {
                                    display: flex;
                                    align-items: flex-end;
                                    font-weight: 600;
                                    em {
                                        color: #62C5FE;
                                        font-size: 26px;
                                    }
                                    span {
                                        color: #666666;
                                        &:before {
                                            content: "/";
                                        }
                                    }
                                }
                                .project-circle-title {
                                    font-size: 12px;
                                    padding-top: 4px;
                                    color: #999999;
                                }
                            }
                        }
                        .project-situation {
                            flex: 1;
                            position: relative;
                            ul {
                                display: flex;
                                flex-direction: column;
                                position: absolute;
                                top: 50%;
                                left: 50%;
                                transform: translate(-50%, -50%);
                                > li {
                                    width: 100%;
                                    color: #BBBBBB;
                                    font-size: 12px;
                                    white-space: nowrap;
                                    display: flex;
                                    align-items: center;
                                    padding: 6px 0;
                                    line-height: 20px;
                                    > em {
                                        padding-left: 14px;
                                        font-size: 18px;
                                        color: #666666;
                                        font-weight: 500;
                                    }
                                }
                            }
                        }
                    }
                    .project-bottom {
                        display: flex;
                        flex-direction: column;
                        border-top: 1px solid #EFEFEF;
                        padding: 18px 0;
                        cursor: default;
                        position: relative;
                        .project-iconbtn {
                            flex: 1;
                            width: 50%;
                            text-align: center;
                            display: flex;
                            align-items: center;
                            padding: 4px 0;
                            &.project-people {
                                width: auto;
                                min-width: 36px;
                                position: absolute;
                                bottom: 18px;
                                right: 0;
                                cursor: pointer;
                                justify-content: flex-end;
                                .userimg-icon,
                                .userimg-count {
                                    width: 36px;
                                    height: 36px;
                                    border-radius: 18px;
                                    margin-left: -16px;
                                    border: 2px solid #ffffff;
                                }
                                .userimg-count {
                                    transform: scale(1);
                                    color: #ffffff;
                                    font-size: 16px;
                                    font-weight: 500;
                                    line-height: 32px;
                                    background-color: #62C5FE;
                                }
                            }
                            .project-iconbtn-icon {
                                font-size: 16px;
                                margin-right: 6px;
                                color: #999;
                            }
                            .project-iconbtn-text {
                                color: #999999;
                                cursor: pointer;
                                &:hover {
                                    color: #0396f2;
                                }
                            }
                        }
                    }
                }
            }
            &:before,
            &:after {
                display: table;
                content: "";
            }
            &:after {
                clear: both;
            }
        }
    }
</style>
<script>
    import Vue from 'vue'
    import TagInput from '../components/TagInput'
    Vue.component('TagInput', TagInput)

    import WContent from "../components/WContent";
    import ProjectArchived from "../components/project/archived";
    import ProjectUsers from "../components/project/users";
    import ProjectStatistics from "../components/project/statistics";
    import ProjectMyFavor from "../components/project/my/favor";
    import ProjectMyJoin from "../components/project/my/join";
    import ProjectMyManage from "../components/project/my/manage";
    import Project from "../mixins/project";
    import WDrawer from "../components/iview/WDrawer";
    export default {
        components: {
            WDrawer,
            ProjectMyManage,
            ProjectMyJoin,
            ProjectMyFavor, ProjectStatistics, ProjectUsers, ProjectArchived, WContent},
        mixins: [
            Project
        ],
        data () {
            return {
                loadIng: 0,

                addShow: false,
                formAdd: {
                    title: '',
                    labels: [],
                    template: 0,
                },
                ruleAdd: {},

                labelLists: [],

                lists: [],
                listPage: 1,
                listTotal: 0,
                listPageSize: 20,

                projectDrawerShow: false,
                projectDrawerTab: 'archived',

                projectListDrawerShow: false,
                projectListDrawerTab: 'myjoin',

                handleProjectId: 0,
            }
        },
        mounted() {
            this.getLists(true);
            //
            $A.setOnTaskInfoListener('pages/project',(act, detail) => {
                let item = this.lists.find((item) => { return item.id == detail.projectid });
                if (!item) {
                    return;
                }
                const persons = detail.persons ? !!detail.persons.find(({username}) => username == this.usrName) : null;
                const unfinishedNum = (add) => {
                    if (add) {
                        item.unfinished++;
                        persons === true && item.self_count++;
                    } else {
                        item.unfinished--;
                        persons === true && item.self_count--;
                    }
                };
                const completeNum = (add) => {
                    if (add) {
                        item.complete++;
                        persons === true && item.self_complete++;
                    } else {
                        item.complete--;
                        persons === true && item.self_complete--;
                    }
                };
                switch (act) {
                    case 'deleteproject':   // 删除项目
                    case 'deletelabel':     // 删除分类
                        this.getLists(true);
                        break;
                    case "create":          // 创建任务
                        unfinishedNum(true);
                        break;
                    case "delete":          // 删除任务
                    case "archived":        // 归档
                        if (detail.complete) {
                            completeNum();
                        } else {
                            unfinishedNum();
                        }
                        break;
                    case "unarchived":      // 取消归档
                        if (detail.complete) {
                            completeNum(true);
                        } else {
                            unfinishedNum(true);
                        }
                        break;
                    case "complete":        // 标记完成
                        completeNum(true);
                        unfinishedNum();
                        break;
                    case "unfinished":      // 标记未完成
                        completeNum();
                        unfinishedNum(true);
                        break;
                }
            }, true);
        },
        deactivated() {
            this.addShow = false;
            this.projectDrawerShow = false;
            this.projectListDrawerShow = false;
        },
        watch: {
            usrName() {
                this.usrLogin && this.getLists(true);
            },
        },
        methods: {
            initLanguage() {
                this.labelLists = [{
                    label: this.$L('空白模板'),
                    value: [],
                }, {
                    label: this.$L('软件开发'),
                    value: [this.$L('产品规划'),this.$L('前端开发'),this.$L('后端开发'),this.$L('测试'),this.$L('发布'),this.$L('其它')],
                }, {
                    label: this.$L('产品开发'),
                    value: [this.$L('产品计划'), this.$L('正在设计'), this.$L('正在研发'), this.$L('测试'), this.$L('准备发布'), this.$L('发布成功')],
                }];
                this.ruleAdd = {
                    title: [
                        { required: true, message: this.$L('请填写项目名称！'), trigger: 'change' },
                        { type: 'string', min: 2, message: this.$L('项目名称至少2个字！'), trigger: 'change' }
                    ]
                };
            },

            setPage(page) {
                this.listPage = page;
                this.getLists();
            },

            setPageSize(size) {
                if (Math.max($A.runNum(this.listPageSize), 20) != size) {
                    this.listPageSize = size;
                    this.getLists();
                }
            },

            getLists(resetLoad) {
                if (resetLoad === true) {
                    this.listPage = 1;
                }
                this.loadIng++;
                $A.apiAjax({
                    url: 'project/lists',
                    data: {
                        page: Math.max(this.listPage, 1),
                        pagesize: Math.max($A.runNum(this.listPageSize), 20),
                    },
                    complete: () => {
                        this.loadIng--;
                    },
                    success: (res) => {
                        if (res.ret === 1) {
                            this.lists = res.data.lists;
                            this.listTotal = res.data.total;
                        }else{
                            this.lists = [];
                            this.listTotal = 0;
                        }
                    }
                });
            },

            addLabels() {
                this.labelsValue = "";
                this.$Modal.confirm({
                    render: (h) => {
                        return h('div', [
                            h('div', {
                                style: {
                                    fontSize: '16px',
                                    fontWeight: '500',
                                    marginBottom: '20px',
                                }
                            }, this.$L('添加流程')),
                            h('TagInput', {
                                props: {
                                    value: this.labelsValue,
                                    autofocus: true,
                                    placeholder: this.$L('请输入流程名称，多个可用英文逗号分隔。')
                                },
                                on: {
                                    input: (val) => {
                                        this.labelsValue = val;
                                    }
                                }
                            })
                        ])
                    },
                    onOk: () => {
                        if (this.labelsValue) {
                            let array = $A.trim(this.labelsValue).split(",");
                            array.forEach((name) => {
                                if ($A.trim(name)) {
                                    this.formAdd.labels.push($A.trim(name));
                                }
                            });
                        }
                    },
                })
            },

            onAdd() {
                this.$refs.add.validate((valid) => {
                    if (valid) {
                        this.loadIng++;
                        $A.apiAjax({
                            url: 'project/add',
                            data: this.formAdd,
                            complete: () => {
                                this.loadIng--;
                            },
                            success: (res) => {
                                if (res.ret === 1) {
                                    this.addShow = false;
                                    this.$Message.success(res.msg);
                                    this.$refs.add.resetFields();
                                    this.$set(this.formAdd, 'template', 0);
                                    //
                                    this.getLists(true);
                                } else {
                                    this.$Modal.error({title: this.$L('温馨提示'), content: res.msg});
                                }
                            }
                        });
                    }
                });
            },

            openComplete(item) {
                if (item.complete > 0) {
                    this.openProject(item.id, item, '已完成')
                } else {
                    this.handleProject('open', item);
                }
            },

            handleProject(event, item) {
                if (item) {
                    this.handleProjectId = item.id;
                }
                switch (event) {
                    case 'favor': {
                        this.favorProject('add', item.id);
                        break;
                    }
                    case 'rename': {
                        this.renameProject(item);
                        break;
                    }
                    case 'transfer': {
                        this.transferProject(item);
                        break;
                    }
                    case 'delete': {
                        this.deleteProject(item.id, () => {
                            this.getLists();
                        });
                        break;
                    }
                    case 'out': {
                        this.outProject(item.id, () => {
                            this.getLists();
                        });
                        break;
                    }

                    case 'open': {
                        this.openProject(item.id, item);
                        break;
                    }
                    case 'archived':
                    case 'member':
                    case 'statistics': {
                        this.projectDrawerShow = true;
                        this.projectDrawerTab = event;
                        break;
                    }
                    case 'myjoin':
                    case 'myfavor':
                    case 'mycreate': {
                        this.projectListDrawerShow = true;
                        this.projectListDrawerTab = event;
                        break;
                    }
                }
            },

            renameProject(item) {
                this.renameValue = "";
                this.$Modal.confirm({
                    render: (h) => {
                        return h('div', [
                            h('div', {
                                style: {
                                    fontSize: '16px',
                                    fontWeight: '500',
                                    marginBottom: '20px',
                                }
                            }, this.$L('重命名项目')),
                            h('Input', {
                                props: {
                                    value: this.renameValue,
                                    autofocus: true,
                                    placeholder: this.$L('请输入新的项目名称')
                                },
                                on: {
                                    input: (val) => {
                                        this.renameValue = val;
                                    }
                                }
                            })
                        ])
                    },
                    loading: true,
                    onOk: () => {
                        if (this.renameValue) {
                            this.$set(item, 'loadIng', true);
                            let title = this.renameValue;
                            $A.apiAjax({
                                url: 'project/rename',
                                data: {
                                    projectid: item.id,
                                    title: title,
                                },
                                complete: () => {
                                    this.$set(item, 'loadIng', false);
                                },
                                error: () => {
                                    this.$Modal.remove();
                                    alert(this.$L('网络繁忙，请稍后再试！'));
                                },
                                success: (res) => {
                                    this.$Modal.remove();
                                    this.$set(item, 'title', title);
                                    setTimeout(() => {
                                        if (res.ret === 1) {
                                            this.$Message.success(res.msg);
                                        } else {
                                            this.$Modal.error({title: this.$L('温馨提示'), content: res.msg});
                                        }
                                    }, 350);
                                }
                            });
                        } else {
                            this.$Modal.remove();
                        }
                    },
                });
            },

            transferProject(item) {
                this.transferValue = "";
                this.$Modal.confirm({
                    render: (h) => {
                        return h('div', [
                            h('div', {
                                style: {
                                    fontSize: '16px',
                                    fontWeight: '500',
                                    marginBottom: '20px',
                                }
                            }, this.$L('移交项目')),
                            h('UserInput', {
                                props: {
                                    value: this.transferValue,
                                    nousername: item.username,
                                    placeholder: this.$L('请输入昵称/用户名搜索')
                                },
                                on: {
                                    input: (val) => {
                                        this.transferValue = val;
                                    }
                                }
                            })
                        ])
                    },
                    loading: true,
                    onOk: () => {
                        if (this.transferValue) {
                            this.$set(item, 'loadIng', true);
                            let username = this.transferValue;
                            $A.apiAjax({
                                url: 'project/transfer',
                                data: {
                                    projectid: item.id,
                                    username: username,
                                },
                                complete: () => {
                                    this.$set(item, 'loadIng', false);
                                },
                                error: () => {
                                    this.$Modal.remove();
                                    alert(this.$L('网络繁忙，请稍后再试！'));
                                },
                                success: (res) => {
                                    this.$Modal.remove();
                                    this.getLists();
                                    setTimeout(() => {
                                        if (res.ret === 1) {
                                            this.$Message.success(res.msg);
                                        } else {
                                            this.$Modal.error({title: this.$L('温馨提示'), content: res.msg});
                                        }
                                    }, 350);
                                }
                            });
                        } else {
                            this.$Modal.remove();
                        }
                    },
                });
            },

            selfProportion(complete, count) {
                if (count <= 0) {
                    return 100;
                }
                return Math.round(complete / count * 100)
            }
        },
    }
</script>
