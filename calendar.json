// INIT

const alt = new Alt();

window.requestAnimFrame = (function(){
  return  window.requestAnimationFrame       ||
          window.webkitRequestAnimationFrame ||
          window.mozRequestAnimationFrame    ||
          function( callback ){
            window.setTimeout(callback, 1000 / 60);
          };
})();



// UTILS

function format(date, format) {
	return date.toString(format);
}

function monthKey(date) {
	return format(date, 'yyyyMM');
}

function dayKey(date) {
  return format(date, 'MMddyyyy');
}

function nextMonth(date) {
	date = new Date(date);
	let next = date.getMonth() + 1;

	let res = new Date(date.setMonth(next % 12));

	if (next > 11) {
		res = new Date(
			res.setFullYear(
				res.getFullYear() + 1
			)
		);
	}

	return res;
}

function prevMonth(date) {
	date = new Date(date);
	let prev = date.getMonth() - 1;
	let res;

	if (prev < 0) {
		res = new Date(date.setMonth(prev + 12));

		res = new Date(
			res.setFullYear(
				res.getFullYear() - 1
			)
		);
	} else {
		res = new Date(date.setMonth(prev));
	}

	return res;
}

function daysOfMonth(date) {
	return [...trailingDays(date), ...currentDays(date), ...leadingDays(date)];
}

function currentDays(date) {
	let days = [];
	let day = new Date(date);
	for (var i = 1; i < 32; i++) {
		day = new Date(day.setDate(i));
		if (!(day.valueOf() && day.getMonth() == date.getMonth())) {
			return days;
		}
		days.push({ value: new Date(day), type: 'current', key: dayKey(day) });
	}

	return days;
}

function trailingDays(date) {
	let days = []
	let day = new Date(date);
	day = new Date(day.setDate(1) - 86400000);
	while (day.getDay() != 6) {
		days.unshift({ value: day, type: 'trailing' });
		day = new Date(day.valueOf() - 86400000);
	}

	return days;
}

function leadingDays(date) {
	let days = []
	let day = new Date(date.setDate(1));
	day = new Date(day.setMonth(day.getMonth() % 12 + 1))
	day = new Date(day.setDate(1));
	while (day.getDay() != 0) {
		days.push({ value: day, type: 'leading' });
		day = new Date(day.valueOf() + 86400000);
	}

	return days;
}


// COMPONENTS

class Calendar extends React.Component {

	constructor(props) {
		super(props);
		this.state = CalendarStore.getState();
	}

	onChange = (state) => {
		this.setState(state);
	};

	componentDidMount() {
		this.setAnimationListener();
		this.scrollToFocus();
		this.windowWidth = window.innerWidth;
		CalendarStore.listen(this.onChange);
	}

	componentWillUpdate(nextProps, nextState) {
		if (nextState.shifted) {
			this.focusMonthOffset = this.focusMonth.offsetTop() - 
				this.refs.root.scrollTop;
		}
	}

	componentDidUpdate() {
		if (this.state.shifted) {
			this.refs.root.scrollTop = this.focusMonth.offsetTop() - 
				this.focusMonthOffset;
		}
	}

	componentWillUnmount() {
		CalendarStore.unlisten(this.onChange);
	}
	
	scrollToFocus() {
		this.refs.root.scrollTop = this.focusMonth.offsetTop() - 32;
	}
	
	checkScroll() {
		if (this.windowWidth !== window.innerWidth) {
			this.scrollToFocus();
			this.windowWidth = window.innerWidth;
		}
	}
	
	setAnimationListener() {
		requestAnimFrame(() => { 
			this.checkScroll(); 
			this.checkFocusMonth();
		});
	}
	
	checkFocusMonth() {
		let focusedTop = this.focusMonth.offsetTop();
		let focusedHeight = this.focusMonth.scrollHeight();
		let scrollTop = this.refs.root.scrollTop;

		if (scrollTop > focusedTop + focusedHeight - 44 + 1) {
			CalendarActions.incrementFocusMonth();
		} else if (scrollTop < focusedTop - 44 - 1) {
			CalendarActions.decrementFocusMonth();
		}
		this.setAnimationListener();		
	}
	
	setRef(ref, idx) {
		if (idx - this.state.focusMonthIdx === 0) { this.focusMonth = ref; }
	}

	render() {
		return (
			<div id='calendar-container' ref="root">
				<CalendarHeader />
				<div id="calendar-body-container">
					{
						this.state.months.map( (month, idx) =>
							<Month days={ month.days }
								month={ month.month }
								today={ this.state.today }
								key={ month.key }
								fromFocus={ idx - this.state.focusMonthIdx }
								ref={ (ref) => { this.setRef(ref, idx); } } />
						)
					}
				</div>
			</div>
		);
	}

}


class CalendarHeader extends React.Component {

	constructor(props) {
		super(props);
		this.days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
	}

	previousMonth() {
		CalendarActions.decrementMonth();
	}

	nextMonth() {
		CalendarActions.incrementMonth();
	}

	render() {
		return (
			<div id="calendar-header-container">
				<div id="calendar-header-days-container">
					<ol id="calendar-header-days">
						{
							this.days.map( (day, idx) =>
								<li className="calendar-header-day" key={ idx }>{ day }</li>
							)
						}
					</ol>
				</div>
			</div>
		);
	}

}


class Month extends React.Component {

	mapWeeks(callback, idx) {
		let start = 0;
		let weeks = [];
		while (this.props.days[start]) {
			weeks.push(
				callback.call(this, this.props.days.slice(start, start + 7), start / 7)
			);
			start += 7;
		}

		return weeks;
	}

	offsetTop() {
		return this.refs.root.offsetTop;
	}

	scrollHeight() {
		return this.refs.root.scrollHeight;
	}
	
	isToday(day) {
		return this.props.today.key === day.key;
	}

	renderWeek(week, idx) {
		return (
			<div className="calendar-week" key={ idx }>
				{ 
					week.map( (day, idx) => 
						<CalendarDay 
							day={ day } key={ idx } isToday={ this.isToday(day) } />
					) 
				}
			</div>
		);
	}

	render() {
		let headerClasses = classNames({
			'calendar-days-header-container': true,
			'active': this.props.fromFocus === 0
		});

		return (
			<div className="calendar-days-container" ref="root">
				<div className={ headerClasses }>
					<h4 className="calendar-days-header primary-header" ref="header">
						<span className="month">{ format(this.props.month, 'MMMM ') }</span>
						<span className="year">{ format(this.props.month, 'yyyy') }</span>
					</h4>
					<h4 className="calendar-days-header scrolling-header" ref="header">
						<span className="month">{ format(this.props.month, 'MMMM ') }</span>
						<span className="year">{ format(this.props.month, 'yyyy') }</span>
					</h4>
				</div>
				<div className="calendar-days-content">
					{ this.mapWeeks( (week, idx) => this.renderWeek(week, idx) ) }
				</div>
			</div>
		);
	}

 }


class CalendarDay extends React.Component {

	isWeekend() {
		let day = this.props.day.value.getDay();
		return day === 0 || day === 6;
	}

	render() {
		let dayClasses = classNames({
			'calendar-day': true,
			[this.props.day.type]: true,
			'weekend': this.isWeekend(),
			'today': this.props.isToday
		});

		return (
			<div className={ dayClasses }>
				<div className='calendar-day-content'>
					{ this.props.day.value.getDate() }
				</div>
			</div>
		);
	}

}

// ACTIONS

class _CalendarActions {

	constructor() {
		this.generateActions(
			'incrementFocusMonth', 'decrementFocusMonth'
		);
	}

}

const CalendarActions = alt.createActions(_CalendarActions);


// STORES


class _CalendarStore {

	constructor() {
		this.bindActions(CalendarActions);
    let date = new Date();
		this.today = { value: date, key: dayKey(date) };

		this.focusMonth = this.constructMonth(new Date());
		this.focusMonthIdx = 5;
		this.months = [this.focusMonth];
		this.pushMonths(5);
		this.unshiftMonths(5);
	}

	onIncrementFocusMonth() {
		this.shifted = false;
		this.focusMonth = this.months[++this.focusMonthIdx];
		if (this.focusMonthIdx > 7) { this.shiftForwards(); }
	}

	onDecrementFocusMonth() {
		this.shifted = false;
		this.focusMonth = this.months[--this.focusMonthIdx];
		if (this.focusMonthIdx < 3) { this.shiftBackwards(); }
	}

	shiftForwards() {
		this.pushMonths(3);
		this.months.splice(0, 3);
		this.focusMonthIdx -= 3;
		this.shifted = true;
	}

	shiftBackwards() {
		this.unshiftMonths(3);
		this.months.splice(10, 3);
		this.focusMonthIdx += 3;
		this.shifted = true;
	}

	constructMonth(date) {
		return {
			key: monthKey(date),
			month: date,
			days: daysOfMonth(date)
		};
	}

	pushMonths(n) {
		for (let i = 0; i < n; i++) {
			this.months.push(
				this.constructMonth(nextMonth(this.months[this.months.length - 1].month))
			);
		}
	}

	unshiftMonths(n) {
		for (let i = 0; i < n; i++) {
			this.months.unshift(
				this.constructMonth(prevMonth(this.months[0].month))
			);
		}
	}

}

const CalendarStore = alt.createStore(_CalendarStore);


// RENDER

React.render(<Calendar />, document.getElementById('app'));
