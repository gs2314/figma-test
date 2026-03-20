<template>
    <div v-if="views.planner" class="row planner no-gutters">
        <div class="col-12 col-sm-12 col-md-6">
            <MobileStepBar v-if="current_max_step >= 2" v-bind="{
                user: user,
                step: step,
                form_data: form_data,
                max_steps: max_steps,
                window_width: window_width,
                current_max_step: current_max_step,
            }" v-on:event="handleEvent"/>

            <div class="trip-planner">
                <div class="row">
                    <span class="planner-title text-center mb-4">
                        {{ translations.planner_title }}
                    </span>
                </div>
                <div class="row my-4 planner-tabs">
                    <div :class="form_data.active_tab === 'one_way' ? 'planner-tab-active' : 'planner-tab'"
                         class="col-6 text-center pointer" @click="switchTab('one_way')">
                        <img class="planner-tab-img" :src="getImageURL('arrow_one_way.svg')" alt="">
                        <span class="tab-title">{{ translations.one_way }}</span>
                    </div>
                    <div :class="form_data.active_tab === 'return' ? 'planner-tab-active' : 'planner-tab'"
                         class="col-6 text-center pointer" @click="switchTab('return')">
                        <img class="planner-tab-img" :src="getImageURL('arrows_return.svg')" alt="">
                        <span class="tab-title">{{ translations.return }}</span>
                    </div>
                </div>
                <div class="row my-4">
                    <div v-show="['one_way', 'return'].includes(form_data.active_tab)">
                        <div class="row p-0">
                            <div class="col-12 p-0 m-0 destinations-inputs" style="position: relative;">
                                <a class="text-decoration-none pointer" data-bs-toggle="modal"
                                   data-bs-target="#destination_modal" @click="setDestinationModalData()">
                                    <div class="input-start">
                                        <img :src="getImageURL('marker.svg')" alt="">
                                        <span class="input-text">
                                            {{ translations.from }}:&nbsp;
                                        </span>
                                        <span class="input-text-secondary">
                                            {{ form_data.from.name ?? translations.island_city }}
                                        </span>
                                        <span v-if="errors.hasOwnProperty('from')"
                                              class="text-decoration-none ms-auto">
                                            <img :src="getImageURL('error.svg')" alt="">
                                        </span>
                                    </div>
                                </a>
                                <div class="swap-destinations-btn pointer" @click="swapDestinations()">
                                    <img :src="getImageURL('arrows_return.svg')" alt="" class="swap-icon">
                                </div>
                                <a class="text-decoration-none pointer" data-bs-toggle="modal"
                                   data-bs-target="#destination_modal" @click="setDestinationModalData('to')">
                                    <div class="input-end">
                                        <img :src="getImageURL('marker.svg')" alt="">
                                        <span class="input-text">
                                            {{ translations.to }}:&nbsp;
                                        </span>
                                        <span class="input-text-secondary">
                                            {{ form_data.to.name ?? translations.island_city }}
                                        </span>
                                        <span v-if="errors.hasOwnProperty('to')"
                                              class="text-decoration-none ms-auto">
                                            <img :src="getImageURL('error.svg')" alt="">
                                        </span>
                                    </div>
                                </a>
                            </div>
                            <div class="col-12 mt-4 p-0 m-0 destinations-inputs">
                                <a class="text-decoration-none pointer" data-bs-toggle="modal"
                                   data-bs-target="#calendar_modal" @click="setCalendarModalField()">
                                    <div :class="form_data.active_tab !== 'return' ? 'single-date' : ''"
                                         class="input-start">
                                        <img :src="getImageURL('calendar.svg')" alt="">
                                        <span class="input-text-secondary">
                                            {{ formatDate(form_data.departure_date) ?? translations.departure_date }}
                                        </span>
                                        <span v-if="errors.hasOwnProperty('departure_date')"
                                              class="text-decoration-none ms-auto">
                                            <img :src="getImageURL('error.svg')" alt="">
                                        </span>
                                    </div>
                                </a>
                                <a v-if="form_data.active_tab === 'return'" class="text-decoration-none pointer"
                                   data-bs-toggle="modal" data-bs-target="#calendar_modal"
                                   @click="setCalendarModalField('return_date')">
                                    <div class="input-end">
                                        <img :src="getImageURL('calendar.svg')" alt="">
                                        <span class="input-text-secondary">
                                            {{ formatDate(form_data.return_date) ?? translations.return_date }}
                                        </span>
                                        <span v-if="errors.hasOwnProperty('return_date')"
                                              class="text-decoration-none ms-auto">
                                            <img :src="getImageURL('error.svg')" alt="">
                                        </span>
                                    </div>
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
                <div v-if="Object.keys(errors).length > 0" class="row mb-4">
                    <div v-for="(error, key) in errors" class="col-12">
                        <span class="input-text-error">
                            {{ error }}
                        </span>
                    </div>
                </div>
                <div class="row mt-4">
                    <div class="col-12 p-0">
                        <a class="booking-details-full text-decoration-none pointer" data-bs-toggle="modal"
                           data-bs-target="#details_modal">
                            <div class="booking-details-item">
                                <img :src="getImageURL('passengers.svg')" alt="">
                                <span class="details-label">{{ translations.adults_children_infants ?? translations.passengers }}</span>
                                <span class="details-quantity-pill">{{ form_data.details.passengers }}</span>
                            </div>
                            <div class="booking-details-separator"></div>
                            <div class="booking-details-item">
                                <img :src="getImageURL('pets.svg')" alt="">
                                <span class="details-label">{{ translations.pets }}</span>
                                <span class="details-quantity-pill">{{ form_data.details.pets }}</span>
                            </div>
                            <div class="booking-details-separator"></div>
                            <div class="booking-details-item">
                                <img :src="getImageURL('vehicles.svg')" alt="">
                                <span class="details-label">{{ translations.vehicles }}</span>
                                <span class="details-quantity-pill">{{ form_data.details.vehicles }}</span>
                            </div>
                        </a>
                    </div>
                </div>
                <div class="row bottom-0">
                    <a class="mt-4 text-decoration-none search-btn" @click="getTrips()">
                        {{ translations.search }}
                    </a>
                </div>
            </div>
        </div>

        <div class="col-12 col-md-6 px-auto">
            <div class="recent">
                <div class="row">
                    <span class="planner-title">
                        {{ translations.book_your_ferry_trips }}
                    </span>
                    <small class="planner-small mt-2">
                        {{ translations.plan_your_trip }}
                    </small>
                </div>
                <div v-if="recent_searches !== null && recent_searches.length > 0" class="row mt-2">
                    <div class="col-12 mb-2">
                        <span class="recent-searches-title">{{ translations.recent_searches }}</span>
                    </div>
                    <div class="col-12">
                        <div v-for="(recent_search, key) in recent_searches"
                             class="recent-search-item mb-3"
                             @click="setRecentSearch(key)">
                            <img :src="getImageURL('marker.svg')" alt="" class="recent-search-icon">
                            <span class="recent-search-item-text">{{ JSON.parse(recent_search)[0].name }}</span>
                            <img v-if="JSON.parse(recent_search)[3] !== null"
                                 :src="getImageURL('arrows_return.svg')" alt="" class="recent-search-arrow">
                            <img v-else :src="getImageURL('arrow_right_smaller.svg')" alt="" class="recent-search-arrow">
                            <span class="recent-search-item-text">{{ JSON.parse(recent_search)[1].name }}</span>
                            <span v-if="JSON.parse(recent_search)[2]" class="recent-search-date ms-auto">
                                {{ formatDate(JSON.parse(recent_search)[2]) }}
                            </span>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="modal" ref="destination_modal" id="destination_modal" tabindex="-1"
             aria-labelledby="destination_modal_label" aria-hidden="true">
            <div class="modal-dialog modal-dialog-scrollable">
                <div class="vue-modal modal-content">
                    <div class="modal-body">
                        <div class="row">
                            <div class="col-12 mb-4">
                                <div class="input">
                                    <img :src="getImageURL('search.svg')" alt="">
                                    <span class="input-text">
                                        {{
                                            form_data.modal_destination === 'from' ? translations.from : translations.to
                                        }}:
                                    </span>
                                    <input type="text" class="search-input input-text-secondary"
                                           :placeholder="translations.island_city" v-model="search"
                                           ref="search_input"
                                           @keyup="filter(0)">
                                </div>
                            </div>
                        </div>

                        <div class="row">
                            <div class="col-12 mb-4">
                                <span v-if="countries.length > 1" class="destination-modal-titles mb-1">
                                    {{ translations.filter_by_country }}
                                </span>

                                <ul v-if="countries.length > 1" :class="countries.length > 3 ? 'flex-column' : ''"
                                    class="nav destination-nav nav-pills">
                                    <li class="nav-item destination-nav-item" v-for="country in countries">
                                        <span @click="filter(country.id)"
                                              :class="country_filter === country.id ? 'active-filter' : ''"
                                              class="pointer country-filter">
                                            {{ country.name }}
                                            <img v-if="country_filter === country.id"
                                                 :src="getImageURL('clear.svg')" alt="">
                                        </span>
                                    </li>
                                </ul>

                                <span class="destination-modal-titles mb-1">
                                    {{ translations.all_ports }} <span v-if="countries.length > 1">({{
                                        selected_country
                                    }})</span>
                                </span>
                                <div class="row mt-2" v-if="ports.length > 0">
                                    <div class="row py-2" v-for="port in ports" @click="selectDestination(port)">
                                        <div class="col pointer text-start">
                                            <img :src="getImageURL('green_dot.svg')" alt="">&nbsp;
                                            <span class="destination-modal-port-name">
                                                {{ port.name }}
                                            </span>
                                        </div>
                                        <div class="col pointer text-end">
                                            <span class="destination-modal-port-country">
                                                {{ getCountryName(port.country_id) }}
                                            </span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <div :class="(!is_loading_modal) ? 'loading d-none' : 'loading'">
                <span class="loading-text">
                    {{ translations.please_wait }}
                </span>
            </div>
        </div>

        <div class="modal" id="details_modal" tabindex="-1"
             aria-labelledby="details_modal_label"
             aria-hidden="true">
            <div class="modal-dialog modal-dialog-scrollable">
                <div class="vue-modal modal-content">
                    <div class="modal-body">
                        <div class="row mb-4">
                            <div v-for="detail in details" class="col-12 py-1 ticket-selection-details">
                                <div class="ticket-selection-row">
                                    <div class="ticket-selection-row-title">
                                        <span>{{ detail.translation }}</span>
                                    </div>
                                    <div class="ticket-selection-row-icons">
                                        <div class="ticket-selection-row-icon">
                                            <img @click="modifyDetails('remove', detail.name)"
                                                 class="pointer"
                                                 :src="getImageURL('remove_smaller.svg')" alt="">
                                        </div>
                                        <div class="ticket-selection-row-quantity">
                                            <span>{{ form_data.details[detail.name] }}</span>
                                        </div>
                                        <div class="ticket-selection-row-icon">
                                            <img @click="modifyDetails('add', detail.name)"
                                                 class="pointer"
                                                 :src="getImageURL('add_smaller.svg')" alt="">
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <CalendarModal v-model:selected_date="form_data[form_data.modal_calendar]"
                       :selected_tab="form_data.active_tab"
                       :form_data="form_data"/>
    </div>

    <div v-if="views.trips" class="row planner no-gutters">
        <TripView v-bind="{
                    user: user,
                    step: step,
                    trips: trips,
                    form_data: form_data,
                    max_steps: max_steps,
                    return_trips: return_trips,
                    window_width: window_width,
                    current_max_step: current_max_step,
                  }" v-on:event="handleEvent"/>
    </div>

    <div v-if="views.seats" class="row planner no-gutters">
        <SeatView v-bind="{
                    user: user,
                    step: step,
                    trips: trips,
                    errors: errors,
                    pricing: pricing,
                    details: details,
                    form_data: form_data,
                    max_steps: max_steps,
                    window_width: window_width,
                    current_max_step: current_max_step,
                  }" v-on:event="handleEvent"/>
    </div>

    <div v-if="views.passenger_details" class="row planner no-gutters">
        <PassengerView v-bind="{
                    user: user,
                    step: step,
                    trips: trips,
                    errors: errors,
                    details: details,
                    pricing: pricing,
                    form_data: form_data,
                    max_steps: max_steps,
                    window_width: window_width,
                    nationalities: nationalities,
                    current_max_step: current_max_step,
                    is_loading_nationalities_modal: is_loading_modal,
                    unfiltered_nationalities: unfiltered_nationalities,
                  }" v-on:event="handleEvent"/>
    </div>

    <div v-if="views.payment" class="row planner justify-content-center">
        <PaymentView v-bind="{
                    user: user,
                    step: step,
                    trips: trips,
                    pricing: pricing,
                    form_data: form_data,
                    max_steps: max_steps,
                    window_width: window_width,
                    current_max_step: current_max_step,
                  }" v-on:event="handleEvent"/>
    </div>

    <div :class="(!is_loading) ? 'loading d-none' : 'loading'">
        <span class="loading-text">
            {{ translations.please_wait }}
        </span>
    </div>

    <div class="modal" id="reset_form_modal" tabindex="-1"
         aria-labelledby="reset_form_modal_label"
         aria-hidden="true" data-bs-backdrop="static">
        <div class="modal-dialog modal-dialog-scrollable">
            <div class="refresh-form modal-content">
                <div class="modal-body w-100">
                    <div class="row">
                        <div class="col-12 d-flex justify-content-center">
                            <img :src="getImageURL('refresh_form_ship.svg')" alt="">
                        </div>
                    </div>
                    <div class="row my-5">
                        <div class="col-12 d-flex mb-4 justify-content-center">
                            <span class="refresh-form-title">{{ translations.get_fresh_results }}</span>
                        </div>
                        <div class="col-12 d-flex mb-4 justify-content-center">
                            <span class="refresh-form-text">{{ translations.get_fresh_results_text }}</span>
                        </div>
                    </div>
                    <div class="row mt-4 bottom-0">
                        <div class="col-12 col-md-6">
                            <a class="text-decoration-none homepage-btn" :href="base_url">
                                {{ translations.homepage }}
                            </a>
                        </div>
                        <div class="col-12 col-md-6">
                            <a class="text-decoration-none refresh-btn pointer" @click="refreshTrips()">
                                {{ translations.refresh }}
                            </a>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div class="modal" ref="error_modal" id="error_modal" tabindex="-1"
         aria-labelledby="error_modal_label"
         aria-hidden="true" data-bs-backdrop="static">
        <div class="modal-dialog modal-dialog-scrollable">
            <div class="refresh-form modal-content">
                <div class="modal-body w-100">
                    <div class="row">
                        <div class="col-12 d-flex justify-content-center">
                            <img :src="getImageURL('error.svg')" alt="">
                        </div>
                    </div>
                    <div class="row my-5">
                        <div class="col-12 d-flex mb-4 justify-content-center">
                            <span class="refresh-form-title">{{ translations.there_was_an_error }}</span>
                        </div>
                        <div class="col-12 d-flex mb-4 justify-content-center">
                            <span class="refresh-form-text">{{
                                    error_modal_message ?? translations.something_went_wrong
                                }}</span>
                        </div>
                    </div>
                    <div class="row mt-4 bottom-0 justify-content-center">
                        <div class="col-12 col-md-6">
                            <a class="text-decoration-none homepage-btn pointer"
                               @click="closeErrorModal()">
                                {{ translations.ok }}
                            </a>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import '/css/vue_styles/trip_planner.css';
import CalendarModal from './CalendarModal.vue';
import TripView from './TripView.vue';
import SeatView from './SeatView.vue';
import PassengerView from './PassengerView.vue';
import PaymentView from './PaymentView.vue';
import MobileStepBar from './MobileStepBar.vue';

export default {
    name: 'TripPlanner',
    components: {
        TripView,
        SeatView,
        PaymentView,
        MobileStepBar,
        CalendarModal,
        PassengerView,
    },
    created() {
        this.emitter.on('handleEvent', (event) => {
            this.handleEvent(event);
        });

        Object.assign(this.form_data.personal_details.passengers, {
            ['passenger_1']: JSON.parse(JSON.stringify(this.details.passengers.default_personal_details))
        });
    },
    mounted() {
        this.user = user;
        this.base_url = base_url;
        this.translations = translations;
        this.get_trips_url = get_trips_url;
        this.get_pricing_url = get_pricing_url;
        this.home_assets_url = home_assets_url;
        this.recent_searches = recent_searches;
        this.get_booking_url = get_booking_url;
        this.save_booking_url = save_booking_url;
        this.get_countries_url = get_countries_url;
        this.get_countries_data_url = get_countries_data_url;
        this.selected_country = this.translations.all_countries;

        this.$nextTick(() => {
            window.addEventListener('resize', this.onResize);
        });

        this.setUserContactDetails();

        let object = this;

        $(this.$refs.destination_modal).on('shown.bs.modal', () => object.$refs.search_input.focus())
        $(this.$refs.destination_modal).on('hidden.bs.modal', this.closeDestinationModal);
    },
    data() {
        return {
            step: 1,
            user: {},
            ports: [],
            trips: [],
            alerts: {},
            search: '',
            errors: {},
            pricing: [],
            booking: [],
            debug: false,
            max_steps: 5,
            base_url: '',
            countries: [],
            translations: {},
            return_trips: [],
            country_filter: 0,
            is_loading: false,
            nationalities: [],
            home_assets_url: '',
            current_max_step: 1,
            recent_searches: [],
            get_booking_url: '',
            unselected_trips: [],
            selected_country: '',
            unfiltered_ports: [],
            get_countries_url: '',
            views: {
                planner: true,
                trips: false,
                seats: false,
                passenger_details: false,
                payment: false,
            },
            error_modal_message: '',
            is_loading_modal: false,
            details: {
                'passengers': {
                    'name': 'passengers',
                    'translation': translations.adults_children_infants,
                    'key': 'passenger_',
                    'default_data': {
                        'type': {},
                        'fare': {},
                        'seat': {},
                    },
                    'default_personal_details': {
                        'gender': '',
                        'first_name': '',
                        'last_name': '',
                        'date_of_birth': '',
                        'nationality': '',
                        'has_island_resident_number': false,
                        'island_resident_number': '',
                    },
                    'contact_details': {
                        'email': '',
                        'country_code': '',
                        'msisdn': '',
                    },
                    'max_number': 9,
                },
                'pets': {
                    'name': 'pets',
                    'translation': translations.pets,
                    'key': 'pet_',
                    'default_data': {
                        'seat': {},
                        'passenger': '',
                        'yen_code': {},
                        'weight': '',
                    },
                    'default_personal_details': {
                        'document_type': {},
                        'document_number': '',
                    },
                    'max_number': 9,
                },
                'vehicles': {
                    'name': 'vehicles',
                    'translation': translations.vehicles,
                    'key': 'vehicle_',
                    'default_data': {
                        'fare': {},
                        'type': {},
                        'passenger': '',
                    },
                    'default_personal_details': {
                        'plate_number': '',
                    },
                    'max_number': 4,
                },
            },
            unselected_return_trips: [],
            unfiltered_nationalities: [],
            window_width: window.innerWidth,
            form_data: {
                to: {},
                from: {},
                return_date: null,
                modal_calendar: '',
                selected_trips: [],
                personal_details: {
                    'passengers': {},
                    'pets': {},
                    'vehicles': {},
                    'contact_details': {
                        'email': '',
                        'country_code': '',
                        'msisdn': '',
                    },
                },
                departure_date: null,
                active_tab: 'one_way',
                modal_destination: '',
                details: {
                    passengers: 1,
                    pets: 0,
                    vehicles: 0,
                },
                total_price: 0,
            },
            passenger_types: ['passengers', 'pets', 'vehicles'],
        }
    },
    methods: {
        logout() {
            this.user = {'logged_in': false, 'data': null};
        },
        swapDestinations() {
            let temp = this.form_data.from;
            this.form_data.from = this.form_data.to;
            this.form_data.to = temp;
        },
        onResize() {
            this.window_width = window.innerWidth
        },
        getTrips() {
            this.invalidatePricing();
            let object = this;

            this.is_loading = true;
            this.form_data.selected_trips = [];
            //this.emptyPassengerDetails();
            this.validateData();

            if (Object.keys(this.errors).length > 0) {
                this.is_loading = false;
                this.scrollToTop();
                return;
            }

            this.trips = [];
            this.return_trips = [];

            axios.post(this.get_trips_url, this.form_data)
                .then(function (results) {
                    if (!results.data.hasOwnProperty('code')) {
                        object.unselected_trips = results.data.trips;
                        object.trips = results.data.trips;

                        if (results.data.return_trips !== null) {
                            object.unselected_return_trips = results.data.return_trips;
                            object.return_trips = results.data.return_trips;
                        }
                    } else {
                        object.showErrorModal(results.data);
                    }
                })
                .finally(() => {
                    object.is_loading = false;

                    if (object.trips === null
                        || Object.keys(object.trips).length === 0) {
                        Object.assign(object.errors, {
                            'no_trips_found': object.translations.no_trips_found,
                        });
                    } else {
                        object.setRefreshTimer();
                        object.current_max_step = 2;
                        object.goTo('trips');
                    }
                })
                .catch(function (err) {
                    object.is_loading = false;
                });
        },
        getBooking() {
            let object = this;

            this.is_loading = true;

            if (Object.keys(this.errors).length > 0) {
                this.is_loading = false;
                this.scrollToTop();
                return;
            }

            object.booking = [];

            axios.post(this.get_booking_url, this.form_data)
                .then(function (results) {
                    if (!results.data['booking'].hasOwnProperty('code')) {
                        object.booking = results.data['booking'];
                    } else {
                        object.showErrorModal(results.data['booking']);
                    }
                })
                .finally(() => {
                    object.is_loading = false;

                    if (!object.booking.hasOwnProperty('code')) {
                        this.goTo('payment');
                    }
                })
                .catch(function (err) {
                    object.is_loading = false;
                });
        },
        getPricing() {
            let object = this;

            this.is_loading = true;
            this.validatePricing();

            if (Object.keys(this.errors).length > 0) {
                this.is_loading = false;
                this.scrollToTop();
                return;
            }

            object.pricing = [];

            axios.post(this.get_pricing_url, this.form_data)
                .then(function (results) {
                    if (!results.data['pricing'].hasOwnProperty('code')) {
                        object.pricing = results.data['pricing'];
                    } else {
                        object.showErrorModal(results.data['pricing']);
                    }
                })
                .finally(() => {
                    object.is_loading = false;
                })
                .catch(function (err) {
                    object.is_loading = false;
                });
        },
        changeStep() {
            if (this.current_max_step <= this.step) {
                this.current_max_step = this.step;
            }

            this.emitToStepBars();
        },
        goToPayment() {
            this.getBooking();
        },
        scrollToTop() {
            window.scrollTo(0,0);
        },
        falsifyViews() {
            this.views.planner = false;
            this.views.trips = false;
            this.views.seats = false;
            this.views.passenger_details = false;
            this.views.payment = false;
        },
        validateData() {
            this.errors = {};

            if (Object.keys(this.form_data.from).length === 0) {
                Object.assign(this.errors, {
                    'from': this.translations.from_error,
                });
            }

            if (Object.keys(this.form_data.to).length === 0) {
                Object.assign(this.errors, {
                    'to': this.translations.to_error,
                });
            }

            if (typeof this.form_data.from.id_or_code !== 'undefined'
                && typeof this.form_data.to.id_or_code !== 'undefined'
                && this.form_data.from.id_or_code === this.form_data.to.id_or_code) {
                Object.assign(this.errors, {
                    'from': this.translations.same_departure_arrival_error,
                    'to': '',
                });
            }

            if (this.form_data.departure_date === null) {
                Object.assign(this.errors, {
                    'departure_date': this.translations.departure_date_error,
                });
            }

            if (this.form_data.active_tab === 'return'
                && this.form_data.return_date === null) {
                Object.assign(this.errors, {
                    'return_date': this.translations.return_date_error,
                });
            }

            if ((this.form_data.return_date !== null && this.form_data.departure_date !== null)
                && new Date(this.form_data.return_date).getDate() < new Date(this.form_data.departure_date).getDate()) {
                Object.assign(this.errors, {
                    'return_date': this.translations.return_before_departure_date_error,
                });
            }

            if (this.form_data.active_tab === 'multi') {
                //TODO: phase 2
            }
        },
        switchTab(tab) {
            this.errors = {};
            this.form_data.active_tab = tab;
        },
        getCountries() {
            let object = this;

            if (Object.keys(object.unfiltered_ports).length > 0) {
                return;
            }

            object.is_loading_modal = true;

            axios.get(this.get_countries_url)
                .then(function (results) {
                    object.countries = results.data.countries;

                    for (const [key, value] of Object.entries(object.countries)) {
                        for (const [port_key, port_value] of Object.entries(value.ports)) {
                            object.unfiltered_ports.push(port_value);
                            object.ports.push(port_value);
                        }
                    }

                    object.ports.sort((a, b) => {
                        if (a.name < b.name) {
                            return -1;
                        }
                        if (a.name > b.name) {
                            return 1;
                        }
                        return 0;
                    });
                    object.unfiltered_ports.sort((a, b) => {
                        if (a.name < b.name)
                            return -1;
                        if (a.name > b.name)
                            return 1;
                        return 0;
                    });

                    object.is_loading_modal = false;
                })
                .catch(function (err) {
                    object.is_loading_modal = false;
                });
        },
        refreshTrips() {
            $('#reset_form_modal').modal('hide');

            this.invalidatePricing();
            this.current_max_step = 1;
            this.goTo('planner');

            if (Object.keys(this.form_data.from).length !== 0
                && Object.keys(this.form_data.to).length !== 0) {
                this.getTrips();
            }
        },
        goTo(location) {
            this.falsifyViews();

            switch (location) {
                case 'planner':
                    this.step = 1;
                    break;
                case 'trips':
                    this.step = 2;
                    break;
                case 'seats':
                    this.step = 3;
                    break;
                case 'passenger_details':
                    this.step = 4;
                    break;
                case 'payment':
                    this.step = 5;
                    break;
                default:
                    break;
            }

            this.views[location] = true;

            this.changeStep();
            this.scrollToTop();
        },
        loggedIn(user) {
            this.user = user;

            this.emitToUnrelatedComponent(['loggedIn', this.user]);
            this.setUserContactDetails();
        },
        addDetail(type) {
            let key = '',
                object = this;

            if (this.form_data.details[type] < this.details[type].max_number) {
                key = this.details[type].key + this.form_data.details[type];
            }

            if (key !== '') {
                this.form_data.selected_trips.map(function (trip) {
                    Object.assign(trip[type], {
                        [key]: JSON.parse(JSON.stringify(object.details[type].default_data))
                    });
                });

                Object.assign(this.form_data.personal_details[type], {
                    [key]: JSON.parse(JSON.stringify(object.details[type].default_personal_details))
                });
            }
        },
        emitToStepBars() {
            this.emitter.emit('steps-bar', {
                'step': this.step,
                'current_max_step': this.current_max_step,
            });
        },
        validatePricing() {
            this.errors = {};
            let object = this,
                drivers = {},
                adults = {};

            this.form_data.selected_trips.forEach(trip => {
                if (!object.errors.hasOwnProperty(trip.id)) {
                    Object.assign(object.errors, {
                        [trip.id]: {},
                    });
                }

                if (!drivers.hasOwnProperty(trip.id)) {
                    drivers[trip.id] = {};
                }

                if (!adults.hasOwnProperty(trip.id)) {
                    adults[trip.id] = 0;
                }

                this.passenger_types.forEach((passenger_type) => {
                    Object.keys(trip[passenger_type]).forEach(key => {
                        Object.keys(object.details[passenger_type].default_data).forEach(detail_key => {
                            if ((typeof trip[passenger_type][key][detail_key] === 'object'
                                    && (Object.keys(trip[passenger_type][key][detail_key]).length === 0))
                                || trip[passenger_type][key][detail_key] === '') {
                                Object.assign(object.errors[trip.id], {
                                    [key + '_' + detail_key]: object.translations[detail_key + '_error'],
                                });
                            }

                            if (passenger_type === 'vehicles' && detail_key === 'passenger') {
                                if (trip[passenger_type][key][detail_key] !== '') {
                                    if (!drivers[trip.id].hasOwnProperty([trip[passenger_type][key][detail_key]])) {
                                        Object.assign(drivers[trip.id], {
                                            [trip[passenger_type][key][detail_key]]: 1,
                                        });
                                    } else {
                                        drivers[trip.id][trip[passenger_type][key][detail_key]]++;
                                    }
                                }
                            }

                            if (passenger_type === 'passengers' && detail_key === 'type') {
                                if (trip.passengers[key].type.key === 'AD') {
                                    if (!adults.hasOwnProperty([trip.id])) {
                                        adults[trip.id] = 1;
                                    } else {
                                        adults[trip.id]++;
                                    }
                                }
                            }
                        });
                    });
                });

                Object.keys(drivers[trip.id]).forEach((key) => {
                    if (drivers[trip.id][key] > 1) {
                        Object.assign(object.errors[trip.id], {
                            ['multi_vehicles_error']: object.translations['multi_vehicles_error'],
                        });
                    }
                });

                Object.keys(adults).forEach((key) => {
                    if (adults[key] <= 0) {
                        Object.assign(object.errors[trip.id], {
                            ['no_adults_error']: object.translations['no_adults_error'],
                        });
                    }
                });
            });

            Object.keys(this.errors).forEach((key) => {
                if (Object.keys(this.errors[key]).length <= 0) {
                    delete this.errors[key];
                }
            });
        },
        formatDate(value) {
            if (value) {
                let date = new Date(value),
                    weekday = date.toLocaleString('default', {weekday: 'short'}),
                    month = date.toLocaleString('default', {month: 'short'}),
                    day = date.toLocaleString('default', {day: 'numeric'});

                return weekday + ', ' + day + ' ' + month;
            }
        },
        setRefreshTimer() {
            /*setTimeout(() => {
                $('#reset_form_modal').modal('show');
            }, 1200000);*/
        },
        pay(payment_type) {
            let object = this,
                post_data = {
                    form_data: this.form_data,
                    booking_data: this.booking,
                    payment_type: payment_type,
                }

            axios.post(this.save_booking_url, post_data)
                .then(function (results) {
                    console.log(results.data);
                })
                .finally(() => {
                    object.is_loading = false;

                    this.getNexiPayHostedPage(payment_type);
                })
                .catch(function (err) {
                    object.is_loading = false;
                });
        },
        closeErrorModal() {
            this.error_modal_message = '';

            $('#error_modal').modal('hide');
        },
        removeDetail(type) {
            let key = this.details[type].key + this.form_data.details[type];

            for (let trip in this.form_data.selected_trips) {
                if (type === 'passengers') {
                    if (Object.keys(this.form_data.selected_trips[trip]['pets']).length > 0) {
                        for (let pet in this.form_data.selected_trips[trip]['pets']) {
                            if (this.form_data.selected_trips[trip]['pets'][pet]['passenger'] === key) {
                                this.form_data.selected_trips[trip]['pets'][pet]['passenger'] = 'passenger_1';
                            }
                        }
                    }

                    if (Object.keys(this.form_data.selected_trips[trip]['vehicles']).length > 0) {
                        for (let vehicle in this.form_data.selected_trips[trip]['vehicles']) {
                            if (this.form_data.selected_trips[trip]['vehicles'][vehicle]['passenger'] === key) {
                                this.form_data.selected_trips[trip]['vehicles'][vehicle]['passenger'] = 'passenger_1';
                            }
                        }
                    }
                }

                delete this.form_data.selected_trips[trip][type][key];
            }

            delete this.form_data.personal_details[type][key];
        },
        getImageURL(image) {
            return this.home_assets_url + image;
        },
        getNationalities() {
            if (Object.keys(this.unfiltered_nationalities).length > 0) {
                return;
            }

            this.is_loading_modal = true;

            let object = this;

            axios.get(object.get_countries_data_url)
                .then(function (results) {
                    object.nationalities = results.data.id_or_codes.map((code, index) => ({
                        id_or_code: code,
                        name: results.data.nationalities[index] ?? ''
                    })).filter(item => item.name !== '');

                    object.nationalities.sort((a, b) => {
                        if (a.name < b.name) {
                            return -1;
                        }
                        if (a.name > b.name) {
                            return 1;
                        }
                        return 0;
                    });

                    object.unfiltered_nationalities = object.nationalities;
                    object.is_loading_modal = false;
                })
                .catch(function (err) {
                    object.is_loading_modal = false;
                });
        },
        invalidatePricing() {
            this.pricing = [];
        },
        showErrorModal(data) {
            this.error_modal_message = data.message;

            $('#error_modal').modal('show');
        },
        validatePassengers() {
            this.errors = {};
            let object = this;

            this.passenger_types.forEach((passenger_type) =>
                Object.keys(object.form_data.selected_trips[0][passenger_type]).forEach(key => {
                    if (!(key in object.form_data.personal_details[passenger_type])) {
                        Object.keys(object.details[passenger_type].default_personal_details).forEach(detail_key => {
                            if (detail_key !== 'island_resident_number') {
                                Object.assign(object.errors, {
                                    [key + '_' + detail_key]: object.translations[detail_key + '_error'],
                                });
                            }
                        });
                    } else {
                        Object.keys(object.details[passenger_type].default_personal_details).forEach(detail_key => {
                            if (detail_key !== 'island_resident_number') {
                                if ((typeof object.form_data.personal_details[passenger_type][key][detail_key] === 'object'
                                        && Object.keys(object.form_data.personal_details[passenger_type][key][detail_key]).length === 0)
                                    || object.form_data.personal_details[passenger_type][key][detail_key] === '') {
                                    Object.assign(object.errors, {
                                        [key + '_' + detail_key]: object.translations[detail_key + '_error'],
                                    });
                                }
                            }
                        });
                    }
                })
            );

            Object.keys(this.details.passengers.contact_details).forEach((detail) => {
                if (this.form_data.personal_details.contact_details[detail] === '') {
                    Object.assign(this.errors, {
                        [detail]: this.translations[detail],
                    });
                }
            });
        },
        handleEvent(payload) {
            switch (payload[0]) {
                case 'selectTrip':
                    this.selectTrip(payload[1], payload[2]);
                    break;
                case 'deselectTrip':
                    this.deselectTrip(payload[1]);
                    break;
                case 'changeDay':
                    this.changeDay(payload[1], payload[2], payload[3]);
                    break;
                case 'modifyDetails':
                    this.modifyDetails(payload[1], payload[2]);
                    break;
                case 'goTo':
                    this.goTo(payload[1]);
                    break;
                case 'setSelectedDetails':
                    this.setSelectedDetails(payload[1], payload[2], payload[3], payload[4], payload[5]);
                    break;
                case 'setPersonalDetails':
                    this.setPersonalDetails(payload[1], payload[2], payload[3], payload[4]);
                    break;
                case 'setContactDetails':
                    this.setContactDetails(payload[1], payload[2]);
                    break;
                case 'getPricing':
                    this.getPricing();
                    break;
                case 'getNationalities':
                    this.getNationalities();
                    break;
                case 'filterNationalities':
                    this.filterNationalities(payload[1]);
                    break;
                case 'goToPayment':
                    this.goToPayment();
                    break;
                case 'logout':
                    this.logout();
                    break;
                case 'loggedIn':
                    this.loggedIn(payload[1]);
                    break;
                case 'setLoggedUserDetails':
                    this.setLoggedUserDetails(payload[1], payload[2], payload[3]);
                    break;
                case 'pay':
                    this.pay(payload[1]);
                    break;
                case 'setTotalPrice':
                    this.setTotalPrice(payload[1]);
                    break;
                default:
                    break;
            }
        },
        setRecentSearch(key) {
            const recent_search = JSON.parse(this.recent_searches[key]);

            this.form_data.from = recent_search[0];
            this.form_data.to = recent_search[1];
            this.form_data.departure_date = recent_search[2];
            this.form_data.return_date = recent_search[3];

            if (this.form_data.return_date !== null) {
                this.switchTab('return');
            }
        },
        deselectTrip(trip_id) {
            if (this.form_data.selected_trips.length > 0 && trip_id !== '') {
                let object = this;

                this.form_data.selected_trips = this.form_data.selected_trips.filter(function (item) {
                    if (item.id === trip_id) {
                        if (item.trip_type === 'one_way') {
                            object.trips = object.unselected_trips;
                        } else {
                            object.return_trips = object.unselected_return_trips;
                        }
                    }

                    return item.id !== trip_id;
                });

                this.current_max_step = this.step;

                this.emitToStepBars();
                //this.emptyPassengerDetails();
            }
        },
        filter(country_id = 0) {
            this.is_loading = true;
            this.ports = this.unfiltered_ports;

            if (typeof this.search !== 'undefined' && this.search !== null && this.search.length > 0) {
                this.country_filter = 0;
                this.selected_country = this.translations.all_countries;
                this.ports = this.ports.filter(item => item.name.toLowerCase().includes(this.search.toLowerCase()));
            }

            if (country_id > 0) {
                if (this.country_filter === country_id) {
                    this.country_filter = 0;
                } else {
                    this.country_filter = country_id;
                    this.ports = this.ports.filter(item => item.country_id === country_id);
                }

                if (this.country_filter > 0) {
                    this.selected_country = this.getCountryName(country_id);
                } else {
                    this.selected_country = this.translations.all_countries;
                }
            }

            this.is_loading = false;
        },
        selectDestination(port) {
            if (this.form_data.modal_destination === 'from') {
                this.form_data.from = port;
                delete this.errors.from;
            } else {
                this.form_data.to = port;
                delete this.errors.to;
            }

            $('#destination_modal').modal('hide');
        },
        closeDestinationModal() {
            if (this.country_filter > 0) {
                this.country_filter = 0;
                this.selected_country = this.translations.all_countries;
            }

            if (typeof this.search !== 'undefined' && this.search !== null && this.search.length > 0) {
                this.search = '';
            }

            this.ports = this.unfiltered_ports;
        },
        emptyPassengerDetails() {
            this.form_data.passengers = {};
            this.form_data.pets = {};
            this.form_data.vehicles = {};
            this.form_data.details = {
                passengers: 1,
                pets: 0,
                vehicles: 0,
            };

            Object.assign(this.form_data.passengers, {
                'passenger_1': {
                    'fare': {},
                    'seat': {},
                }
            });
        },
        setUserContactDetails() {
            if (this.user.hasOwnProperty('logged_in') && this.user.logged_in) {
                let contact_details = {
                    'email': this.user.data.email,
                    'country_code': this.user.data.country_code,
                    'msisdn': this.user.data.msisdn,
                };

                Object.assign(this.form_data.personal_details.contact_details, contact_details);
            }
        },
        getCountryName(country_id) {
            let country_name = this.countries.filter(item => [country_id].includes(item.id))
                .map(item => {
                    return item.name
                });

            return country_name[0];
        },
        setTotalPrice(total_price) {
            this.form_data.total_price = total_price;
        },
        modifyDetails(action, type) {
            this.invalidatePricing();
            if (action === 'add') {
                if (type === 'vehicles' && this.form_data.details[type] < 4
                    || (type !== 'vehicles' && this.form_data.details[type] < 9)) {
                    this.form_data.details[type]++;
                }

                if (Object.keys(this.form_data.selected_trips).length > 0) {
                    this.addDetail(type);
                }
            } else {
                if ((type === 'passengers' && this.form_data.details[type] >= 2)
                    || (type !== 'passengers' && this.form_data.details[type] > 0)) {
                    if (Object.keys(this.form_data.selected_trips).length > 0) {
                        this.removeDetail(type);
                    }

                    this.form_data.details[type]--;
                }
            }
        },
        filterNationalities(search) {
            this.is_loading = true;
            this.nationalities = this.unfiltered_nationalities;

            if (search.length > 0) {
                this.nationalities = this.nationalities.filter(
                    item => {
                        return item.name.toLowerCase().includes(search.toLowerCase());
                    }
                );
            }

            this.is_loading = false;
        },
        setContactDetails(type, data) {
            this.form_data.personal_details.contact_details[type] = data;
        },
        filterTrips(trip_id, trip_type) {
            let current_trip = {},
                new_details = {
                    'passengers': {},
                    'pets': {},
                    'vehicles': {},
                };

            if (trip_type === 'one_way') {
                this.trips = this.trips.filter((item) => item.id === trip_id);
                current_trip = this.trips[0];
            }

            if (trip_type === 'return') {
                this.return_trips = this.return_trips.filter((item) => item.id === trip_id);
                current_trip = this.return_trips[0];
            }

            if (this.form_data.selected_trips.indexOf(current_trip) === -1) {
                for (let [detail_key, detail_value] of Object.entries(this.form_data.details)) {
                    let key = this.details[detail_key].key;

                    if (detail_value > 0) {
                        for (let i = 1; i <= detail_value; i++) {
                            Object.assign(new_details[detail_key], {
                                [key + i]: JSON.parse(JSON.stringify(this.details[detail_key].default_data)),
                            });
                        }
                    }
                }

                current_trip['trip_type'] = trip_type;

                Object.assign(
                    current_trip,
                    JSON.parse(JSON.stringify(new_details))
                )

                if (trip_type === 'return') {
                    this.form_data.selected_trips.push(current_trip);
                } else {
                    this.form_data.selected_trips.splice(
                        0,
                        0,
                        current_trip
                    );
                }
            }
        },
        emitToUnrelatedComponent(payload) {
            this.emitter.emit('handleLoggIn', payload);
        },
        getNexiPayHostedPage(payment_type) {

        },
        setLoggedUserDetails(details, key, type) {
            if (Object.keys(this.unfiltered_nationalities).length === 0) {
                this.getNationalities();
            }

            if (typeof this.form_data.personal_details[type][key] === 'undefined') {
                Object.assign(this.form_data.personal_details[type], {
                    [key]: {},
                });
            }

            Object.assign(this.form_data.personal_details[type][key],
                JSON.parse(JSON.stringify(details)));
        },
        selectTrip(trip_id, trip_type = 'one_way') {
            this.invalidatePricing()
            this.is_loading = true;

            if (trip_id !== '') {
                this.filterTrips(trip_id, trip_type);
            }

            this.is_loading = false;
        },
        setDestinationModalData(modal_type = 'from') {
            this.form_data.modal_destination = modal_type;
            this.getCountries();
        },
        setCalendarModalField(modal_type = 'departure_date') {
            this.form_data.modal_calendar = modal_type;
        },
        setPersonalDetails(passenger_key, detail_type, data, type) {
            if (typeof this.form_data.personal_details[type][passenger_key] === 'undefined') {
                Object.assign(this.form_data.personal_details[type], {
                    [passenger_key]: JSON.parse(JSON.stringify(this.details[type].default_personal_details)),
                });
            }

            if (typeof data === 'object' && !Array.isArray(data) && data !== null) {
                Object.assign(this.form_data.personal_details[type][passenger_key][detail_type], data);
            } else {
                this.form_data.personal_details[type][passenger_key][detail_type] = data;

                if (detail_type === 'has_island_resident_number' && !data) {
                    this.form_data.personal_details[type][passenger_key].island_resident_number = '';
                }
            }
        },
        changeDay(value, next = false, date_type = 'departure_date') {
            if (value) {
                let date = new Date(value),
                    now = new Date();

                if (next) {
                    date.setDate(date.getDate() + 1);
                } else {
                    date.setDate(date.getDate() - 1);
                }

                if (date.getDate() >= now.getDate()) {
                    let day = date.toLocaleString('default', {day: 'numeric'}).padStart(2, '0'),
                        month = date.toLocaleString('default', {month: 'numeric'}).padStart(2, '0'),
                        year = date.toLocaleString('default', {year: 'numeric'});

                    this.form_data[date_type] = year + '-' + month + '-' + day;

                    this.getTrips();

                    this.current_max_step = this.step;

                    this.emitToStepBars();
                }
            }
        },
        setSelectedDetails(passenger_key, detail_type, data, type, selected_trip_key) {
            this.invalidatePricing();
            if (typeof data === 'object' && !Array.isArray(data) && data !== null) {
                Object.assign(this.form_data.selected_trips[selected_trip_key][type][passenger_key][detail_type], data);
            } else {
                this.form_data.selected_trips[selected_trip_key][type][passenger_key][detail_type] = data;
            }
        },
    },
    beforeUnmount() {
        window.removeEventListener('resize', this.onResize);
    }
};
</script>
