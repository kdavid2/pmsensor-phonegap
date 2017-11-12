var myApp = new Framework7({
    modalTitle: 'Logan Admin Portal',
    animateNavBackIcon: true
});
$.ajaxSetup({
    data: {
        "token": $.jStorage.get("token"),
    },
    type: "POST"
});
// Expose Internal DOM library
var $$ = Framework7.$;
var BASE_URL = 'http://hzhou.me:8888/service/';
// Add main view
var mainView = myApp.addView('.view-main', {
    // Enable Dynamic Navbar for this view
    dynamicNavbar: true
});
// Add another view, which is in right panel
var rightView = myApp.addView('.view-right', {
    // Enable Dynamic Navbar for this view
    dynamicNavbar: true
});
if (!$.jStorage.get("token")) {
    mainView.loadPage('login.html');
}
// Show/hide preloader for remote ajax loaded pages
// Probably should be removed on a production/local app
$$(document).on('ajaxStart', function() {
    myApp.showIndicator();
});
$$(document).on('ajaxComplete', function() {
    myApp.hideIndicator();
});
//document.location = 'http://google.com';
// Events for specific pages when it initialized
$$(document).on('pageInit', function(e) {
    var page = e.detail.page;
    // Handle Modals Page event when it is init
    remoteURL = '';
    type = 'occupancy';
    isPrediction = 0;
    fromdate = myApp.getDate(-30); // Chart data from the first day of current year
    todate = myApp.getDate(); // Chart data to today

    if (page.name === 'login') {
        $$('.loginBtn').on('click', function() {
            var name = $('#name').val();
            var password = $('#password').val();
            myApp.login(name, password);
        });
    }
    if (page.name === 'centralWest') {
        remoteURL = BASE_URL + 'garage?garage=centralWest&';
    } else if (page.name === 'terminalB') {
        remoteURL = BASE_URL + 'garage?garage=terminalB&';
    } else if (page.name === 'terminalE') {
        remoteURL = BASE_URL + 'garage?garage=terminalE&';
    } else if (page.name === 'economyLot') {
        remoteURL = BASE_URL + 'garage?garage=economyLot&';
    }
    myApp.loadChart();
    if (page.name === 'centralWest' || page.name === 'terminalB' || page.name === 'terminalE' || page.name === 'economyLot') {
        $$('.action-period').on('click', function() {
            myApp.updatePeriod();
            var buttons = [{
                fromdate: true
            }, {
                todate: true
            }, {
                text: 'Get Chart',
                onClick: function() {
                    fromdate = $('.from-date').val();
                    todate = $('.to-date').val();
                    myApp.loadChart(fromdate, todate);
                }
            }, ];
            myApp.myactions(buttons);
        });
        $$('.action-type').on('click', function() {
            myApp.updatePeriod();
            var buttons = [{
                text: 'Occupancy',
                onClick: function() {
                    type = 'occupancy';
                    myApp.loadChart();
                }
            }, {
                text: 'Entry',
                onClick: function() {
                    type = 'entry';
                    myApp.loadChart();
                }
            }, {
                text: 'Exits',
                onClick: function() {
                    type = 'exits';
                    myApp.loadChart();
                }
            }, {
                text: 'Duration',
                onClick: function() {
                    type = 'duration';
                    myApp.loadChart();
                }
            }, ];
            myApp.myactions(buttons);
        });
        $$('.history-selected').on('click', function() {
            myApp.updatePeriod();
            isPrediction = 0;
            myApp.loadChart();
            myApp.closePanel();
        });
        $$('.predicton-selected').on('click', function() {
            myApp.updatePeriod();
            isPrediction = 1;
            myApp.loadChart();
            myApp.closePanel();
        });
        $$('.logout-selected').on('click', function() {
            myApp.logout();
        });
    }
});
$$('.pre-loader').on('click', function() {
    myApp.showPreloader();
})
// Required for demo popover
$$('.popover a').on('click', function() {
    myApp.closeModal('.popover');
});
// Change statusbar bg when panel opened/closed
$$('.panel-left').on('open', function() {
    $$('.statusbar-overlay').addClass('with-panel-left');
});
$$('.panel-right').on('open', function() {
    $$('.statusbar-overlay').addClass('with-panel-right');
});
$$('.panel-left, .panel-right').on('close', function() {
    $$('.statusbar-overlay').removeClass('with-panel-left with-panel-right');
});

function genChartByHighCharts(_chartData, _elementId, _title, _label_title, _valueSuffix) {
    myApp.hidePreloader();
    $('#' + _elementId).empty();
    Highcharts.setOptions({
        global: {
            timezoneOffset: 4 * 60
        }
    });
    $('#' + _elementId).highcharts('StockChart', {
        chart: {
            resetZoomButton: {
                theme: {
                    display: 'none'
                }
            },
            events: {
                load: function() {
                    if (isPrediction == 0) {
                        this.xAxis[0].setExtremes(Date.parse(fromdate), Date.parse(todate));
                    }
                }
            }
        },
        rangeSelector: {
            selected: 2,
            inputEnabled: false,
            buttons: [{
                type: 'day',
                count: 1,
                text: '1d'
            }, {
                type: 'week',
                count: 1,
                text: '1w'
            }, {
                type: 'month',
                count: 1,
                text: '1m'
            }, {
                type: 'month',
                count: 6,
                text: '6m'
            }, {
                type: 'year',
                count: 1,
                text: '1y'
            }, {
                type: 'all',
                text: 'All'
            }],
        },
        title: {
            text: _title
        },
        series: [{
            name: _label_title,
            data: _chartData,
            tooltip: {
                valueDecimals: 2,
                valueSuffix: _valueSuffix
            }
        }],
        xAxis: {
            minRange: 6 * 3600 * 1000 // Do not allow user to zoom in less than 6 hours
        },
        exporting: {
            enabled: false
        }
    });
}
myApp.loadChart = function(firstDate, lastDate) {
    firstDate = typeof firstDate !== 'undefined' ? firstDate : myApp.getDate(-365); // Chart data from the first day of current year
    lastDate = typeof lastDate !== 'undefined' ? lastDate : myApp.getDate(); // Chart data to today
    if (isPrediction) {
        if (type == 'duration') {
            // alert: there is no prediction chart for duration
            myApp.alert("There is no prediction chart for duration");
        } else {
            $.getJSON(remoteURL + 'type=' + type + '&isPrediction=' + isPrediction).done(function(data) {
                genChartByHighCharts(data, 'chart-content', type, type, '');
            });
        }
    } else if (type == 'duration') {
        $.getJSON(remoteURL + 'type=' + type + '&from=' + firstDate + '&to=' + lastDate).done(function(data) {
            genChartByHighCharts(data, 'chart-content', type, type, ' min');
        });
    } else {
        $.getJSON(remoteURL + 'type=' + type + '&from=' + firstDate + '&to=' + lastDate).done(function(data) {
            genChartByHighCharts(data, 'chart-content', type, type, '');
        });
    }
};
myApp.updateChart = function() {
    var chart = $('#chart-content').highcharts();
    chart.xAxis[0].setExtremes(Date.parse(fromdate), Date.parse(todate));
};
myApp.getDate = function(dayOffSet) {
    if (dayOffSet == undefined) {
        dayOffSet = 0;
    }
    var today = new Date();
    var milli = (new Date()).getMilliseconds() + dayOffSet * 24 * 3600 * 1000;
    today.setMilliseconds(milli);
    var dd = today.getDate();
    var mm = today.getMonth() + 1; //January is 0!
    var yyyy = today.getFullYear();
    if (dd < 10) {
        dd = '0' + dd
    }
    if (mm < 10) {
        mm = '0' + mm
    }
    today = yyyy + '-' + mm + '-' + dd;
    return today;
};
myApp.updatePeriod = function() {
    if (isPrediction == 0) {
        var chart = $('#chart-content').highcharts();
        fromdate = myApp.translateDate(chart.xAxis[0].getExtremes().min);
        todate = myApp.translateDate(chart.xAxis[0].getExtremes().max);
    }
};
myApp.translateDate = function(milli) {
    var today = new Date(milli);
    var dd = today.getDate();
    var mm = today.getMonth() + 1; //January is 0!
    var yyyy = today.getFullYear();
    if (dd < 10) {
        dd = '0' + dd
    }
    if (mm < 10) {
        mm = '0' + mm
    }
    today = yyyy + '-' + mm + '-' + dd;
    return today;
};
myApp.myactions = function(params) {
    _modalTemplateTempDiv = document.createElement('div');
    params = params || [];
    if (params.length > 0 && !$.isArray(params[0])) {
        params = [params];
    }
    var actionsTemplate = myApp.params.modalActionsTemplate;
    var buttonsHTML = '';
    for (var i = 0; i < params.length; i++) {
        for (var j = 0; j < params[i].length; j++) {
            if (j === 0) buttonsHTML += '<div class="actions-modal-group">';
            var button = params[i][j];
            var buttonClass = button.label ? 'actions-modal-label' : 'actions-modal-button';
            if (button.bold) buttonClass += ' actions-modal-button-bold';
            if (button.red) buttonClass += ' actions-modal-button-red';
            if (button.fromdate) {
                buttonsHTML += '<span class="actions-modal-label"><input type="date" min="2014-05-30" class="list-button item-link from-date" name="from-date" value="' + fromdate + '"></span>';
            } else if (button.todate) {
                buttonsHTML += '<span class="actions-modal-label"><input type="date" min="2014-06-01" class="to-date" name="to-date" value="' + todate + '"></span>';
            } else if (button.text) {
                buttonsHTML += '<span class="' + buttonClass + '">' + button.text + '</span>';
            }
            if (j === params[i].length - 1) buttonsHTML += '</div>';
        }
    }
    var modalHTML = actionsTemplate.replace(/{{buttons}}/g, buttonsHTML);
    _modalTemplateTempDiv.innerHTML = modalHTML;
    var modal = $(_modalTemplateTempDiv).children();
    $('body').append(modal[0]);
    var groups = modal.find('.actions-modal-group');
    groups.each(function(index, el) {
        var groupIndex = index;
        $(el).children().each(function(index, el) {
            var buttonIndex = index;
            var buttonParams = params[groupIndex][buttonIndex];
            if ($(el).hasClass('actions-modal-button')) {
                $(el).on('click', function(e) {
                    if (buttonParams.close !== false) myApp.closeModal(modal);
                    if (buttonParams.onClick) buttonParams.onClick(modal, e);
                });
            }
        });
    });
    myApp.openModal(modal);
    return modal[0];
};
myApp.login = function(name, password) {
    $.post(BASE_URL + "login", {
        name: name,
        password: password
    }, function(result) {
        console.log(result);
        if (result.status == 200) {
            $.jStorage.set("token", result.token);
            // store name and password to local storage
            $.jStorage.set("name", name);
            $.jStorage.set("password", password);
            document.location = 'index.html';
        } else {
            myApp.alert('cannot find the user');
            $.jStorage.deleteKey("name");
            $.jStorage.deleteKey("password");
            $('#name').val("");
            $('#password').val("");
            return;
        }
    });
};
myApp.logout = function() {
    myApp.closePanel();
    mainView.loadPage('login.html');
    $.jStorage.deleteKey("name");
    $.jStorage.deleteKey("token");
    $.jStorage.deleteKey("password");
    //document.location = 'index.html';
};