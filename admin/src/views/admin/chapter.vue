<template>
    <div>
        <p>
            <button v-on:click="add()" class="btn btn-white btn-default btn-round">
                <i class="ace-icon fa fa-edit"></i>
                New
            </button>
            &nbsp;
            <button v-on:click="list(1)" class="btn btn-white btn-default btn-round">
                <i class="ace-icon fa fa-refresh"></i>
                Refresh
            </button>

        </p>

        <pagination ref="pagination" v-bind:list="list" v-bind:itemCount="8"> </pagination>
        <table id="simple-table" class="table  table-bordered table-hover">
        <thead>
        <tr>
            <th>ID</th>
            <th>Title</th>
            <th>Course ID</th>
            <th>Operation</th>
        </tr>
        </thead>

        <tbody>
        <tr v-for="chapter in chapters">
            <td>{{chapter.id}}</td>
            <td>{{chapter.name}}</td>
            <td>{{chapter.courseId}}</td>


            <td>
                <div class="hidden-sm hidden-xs btn-group">

                    <button v-on:click="edit(chapter)"  class="btn btn-xs btn-info">
                        <i class="ace-icon fa fa-pencil bigger-120"></i>
                    </button>

                    <button v-on:click="del(chapter.id)" class="btn btn-xs btn-danger">
                        <i class="ace-icon fa fa-trash-o bigger-120"></i>
                    </button>

                </div>
            </td>
        </tr>

        </tbody>
    </table>
        <div id="form-modal" class="modal fade" tabindex="-1" role="dialog">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
                        <h4 class="modal-title">New</h4>
                    </div>
                    <div class="modal-body">
                        <form class="form-horizontal">
                            <div class="form-group">
                                <label class="col-sm-2 control-label">Title</label>
                                <div class="col-sm-10">
                                    <input v-model="chapter.name" class="form-control" placeholder="Title">
                                </div>
                            </div>
                            <div class="form-group">
                                <label class="col-sm-2 control-label">Course ID</label>
                                <div class="col-sm-10">
<!--                                    <p class="form-control-static">{{course.name}}</p>-->
                                    <input v-model="chapter.courseId" class="form-control" placeholder="Course ID">
                                </div>
                            </div>
                        </form>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-default" data-dismiss="modal">Cancel</button>
                        <button v-on:click="save()" type="button" class="btn btn-primary">Save</button>
                    </div>
                </div><!-- /.modal-content -->
            </div><!-- /.modal-dialog -->
        </div><!-- /.modal -->
    </div>
</template>

<script>
    import Pagination from "../../components/pagination";
    export default {
        name: 'chapter',
        components: {Pagination},
        data: function() {
            return {
                chapter: {},
                chapters: []
            }
        },
        mounted: function(){
            let _this = this;
            _this.$refs.pagination.size = 5;
            _this.list(1);
            // this.$parent.activeSidebar("business-chapter-sidebar");
        },
        methods: {
            add() {
                let _this = this;
                _this.chapter = {};
                $("#form-modal").modal("show");
            },

            edit(chapter) {
                let _this = this;
                _this.chapter = $.extend({}, chapter);
                $("#form-modal").modal("show");
            },

            list(page) {
                let _this = this;
                _this.$ajax.post('http://127.0.0.1:9000/business/admin/chapter/list',{
                    page: page,
                    size: _this.$refs.pagination.size,
                }).then((response)=>{
                    let resp = response.data;
                    _this.chapters = resp.content.list;
                    _this.$refs.pagination.render(page, resp.content.total)
                })
            },

            save(page) {
                let _this = this;

                // 保存校验
                if (!Validator.require(_this.chapter.name, "name")
                    || !Validator.length(_this.chapter.courseId, "course ID", 1, 8)) {
                    return;
                }
                // _this.chapter.courseId = _this.course.id;

                Loading.show();
                _this.$ajax.post('http://127.0.0.1:9000/business/admin/chapter/save', _this.chapter).then((response)=>{
                    Loading.hide();
                    let resp = response.data;
                    if (resp.success) {
                        $("#form-modal").modal("hide");
                        _this.list(1);
                        Toast.success("save success!");
                    } else {
                        Toast.warning(resp.message)
                    }
                })
            },

            del(id) {
                let _this = this;
                Confirm.show("Data deleted can't be retrieved. Confirm delete？", function () {
                    Loading.show();
                    _this.$ajax.delete('http://127.0.0.1:9000/business/admin/chapter/delete/' + id).then((response)=>{
                        Loading.hide();
                        let resp = response.data;
                        if (resp.success) {
                            _this.list(1);
                            Toast.success("delete success！");
                        }
                    })
                });
            },


        }
    }
</script>