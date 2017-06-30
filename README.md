# Front-End-
Front-End code samples

@model Deploy.Web.Models.ViewModels.BaseViewModel
@{
    ViewBag.Title = "IndexNG";
    Layout = "~/Views/Shared/_BlogLayout.cshtml";
}

<style>
    h2 {
        text-align: center;
        
    h4 {
        text-align: center;
    }

    .sidebar-search {
        margin-top: 20px;
    }

    .btnCreate { 
        margin-bottom: 65px;
    }
</style>

<!-- begin #content -->
<div id="content" class="content" ng-controller="squadsController as sqVM">
    <!-- begin container -->
    <div class="container">
        <ol class="breadcrumb">
            <li><a href="/">Home</a></li>
            <li class="active">Squads</li>
        </ol>
        <h1 class="page-header text-center">Squads@*<small> Browse existing squads below</small>*@</h1>
        <h4>Squads are community groups organized by special interests <br /> to promote mentorship and collaborative learning. </h4>
        <!-- begin row -->
        <div class="row">
            <!-- begin col-9 -->
            <div id="squadIndex">

                <div class="row section-container">

                    @* **SEARCH** *@
                    <div class="col-md-8 col-xs-12">
                        <div class="input-group sidebar-search pull-left">
                            <input type="text" class="form-control" ng-model="sqVM.searchString" placeholder="Search Squads..." />
                            <span class="input-group-btn">
                                <button ng-click="sqVM.search()" class="btn btn-inverse" type="button"><i class="fa fa-search"></i></button>
                            </span>
                        </div>
                    </div>
                    <div class="col-md-4">
                        <a ng-click="sqVM.create()" class="btn btn-success btn-xs read-btn pull-right btnCreate m-t-20">Create New Squad </a>
                    </div>
                </div>
                <!-- begin post-list -->
                <div class="post-list post-grid post-grid-3">
                    <div class="post-li pull-left" ng-repeat='item in sqVM.filteredItems'>
                        <!-- begin post-content -->
                        <div class="post-content">
                            <!-- begin post-info -->
                            <div class="post-info">
                                <h4 class="post-title">
                                    <a href="#" ng-click="sqVM.goHome(item)">{{item.name}}</a>
                                </h4>
                                <div class="post-desc">
                                    <p ng-bind-html="sqVM.sanitize(item.mission)">{{item.mission}}</p>
                                    <br />
                                    {{item.notes}}
                                </div>
                                <div class="btn read-btn-container">
                                    <a ng-click="sqVM.select(item)" class="btn btn-primary btn-xs read-btn ">Read More <i class="fa fa-angle-double-right"></i></a>
                                </div>
                            </div>
                            <!-- end post-info -->
                        </div>
                        <!-- end post-content -->
                    </div>
                </div>
                <!-- end #content -->
                <div class="col-md-12 text-center">
                    <ul uib-pagination ng-model="sqVM.currentPage" total-items="sqVM.totalItems" boundary-link="true" ng-change="sqVM.pageChanged() " items-per-page="sqVM.itemsPerPage"></ul>
                </div>
            </div>
        </div>
    </div>
</div>


@section scripts {
    <script src="~/Scripts/deploy.services.squad.js"></script>

    <script>

        (function () {
            "use strict";
            angular.module(APPNAME)
                 .factory('squadService', SquadService);

            SquadService.$inject = ['$baseService', '$deploy'];    

            function SquadService($baseService, $deploy) {
                var aDeployServiceObject = deploy.services.squad;
                // Simlates inheritance, giving access to $window and $location services and the getNotifier function
                // to our new service.
                var newService = $baseService.merge(true, {}, aDeployServiceObject, $baseService);
                return newService;
            }

        })();

        (function () { // IIFE -- Immediately Invoked Function Expression, provides context for all the code within it to execute
            "use strict";
            angular.module(APPNAME)
            .controller('squadsController', SquadsController); // This is where we register the controller

            SquadsController.$inject = ['$scope', '$window', '$baseController', 'squadService', '$sce']; // Dependency Injection

            // Constructor below
            function SquadsController($scope, $window, $baseController, squadService, $sce) {

                // Administrative stuff
                var vm = this;
                vm.$scope = $scope;
                // Simulate inheritance to add
                // $document, $log, $route, $routeParams, $systemEventService, $alertService, $deploy services
                // to our controller.
                $baseController.merge(vm, $baseController);
                vm.squadService = squadService;
                vm.$window = $window;
                vm.sanitize = _sanitize;

                // ViewModel
                vm.goHome = _goHome;
                vm.items = [];
                vm.currentPage = 1;
                vm.itemsPerPage = 6; //items per page
                vm.totalItems = 40;
                vm.item = null;  // copy of item being edited
                vm.itemIndex = -1; // index of item being edited
                vm.select = _select;
                vm.add = _add;
                vm.create = _create;
                vm.pageChanged = _pageChanged;
                vm.search = _search;
                vm.edit = _edit;
                vm.items.filter =
                vm.filteredItems = []

                //vm.message = "string";

                // "The fold"

                _render();

                function _render() {
                    vm.squadService.getAll(_getAllSuccess, _getAllError);
                }

                // PAGINATION
                function _pageChanged() {
                    var begin = ((vm.currentPage - 1) * vm.itemsPerPage);
                    var end = begin + vm.itemsPerPage;
                    if (vm.items) {
                        vm.filteredItems = vm.items.slice(begin, end);
                    }
                    else {
                        vm.filteredItems = [];
                    }
                    console.log("vm.filteredItems", vm.filteredItems);
                    console.log("vm.currentPage", vm.currentPage);
                    console.log("vm.itemsPerPage", vm.itemsPerPage);
                }

                function _getAllSuccess(data) {
                    vm.$scope.$apply(function () {
                        vm.items = data.items;
                    });
                    _pageChanged();
                    vm.$alertService.success("Retrieved All Squads");
                }

                function _getAllError(jqXHR) {
                    vm.$alertService.error(jqXHR.responseText, "GetAll failed");
                }


                function _goHome(data) {
                    vm.itemIndex = vm.items.indexOf(data);
                    vm.$window.location.href = '/squads/' + data.id + '/homeNg1/';
                }

                function _select(foo) {
                    vm.itemIndex = vm.items.indexOf(foo);
                    //vm.squadService.getById(l.id, _getByIdSuccess, _onError)
                    vm.$window.location.href = '/squads/' + foo.id + '/homeNg1/';
                }

                function _edit(foo) {
                    vm.itemIndex = vm.items.indexOf(foo);
                    vm.$window.location.href = "/squads/" + foo.id + "/manage/";
                }

                function _create(create) {
                    vm.itemIndex = vm.items.indexOf(create);
                    vm.$window.location.href = "/squads/manage";
                }

                function _getByIdSuccess(data) {
                    if (data.item) {
                        vm.$scope.$apply(function () {
                            vm.item = data.item;
                        });
                    }
                }

                function _onError(jqXHR) {
                    vm.alertService.error(jqXHR.responseText, "GetById failed");
                }

                function _add() {
                    vm.item = {};
                    vm.itemIndex = -1;
                }

                // SEARCH
                function _search() {
                    vm.squadService.search(vm.searchString, _onSearchSuccess, _onSearchError)
                }

                function _onSearchSuccess(data) {
                    vm.items = [];
                    vm.totalItems = data.items.length;
                    vm.$scope.$apply(function () {
                        vm.items = data.items;
                        _pageChanged();
                    });

                }

                function _onSearchError(jqXHR) {

                    vm.$alertService.error(jqXHR.responseText, "Search failed");
                }
                function _sanitize(html_code) {
                    return $sce.trustAsHtml(html_code);
                };

            }
        })();


    </script>

}

