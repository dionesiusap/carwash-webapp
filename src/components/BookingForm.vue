<template>
    <div class="bookingform">
        <div class="hero">
            <div class="hero-body">
                <div class="container">
                    <div class="columns is-centered">
                        <div class="column is-10">
                            <div class="columns full-form">
                                <div class="column is-two-fifths is-hidden-mobile decoration-column">
                                    <!-- xxx -->
                                </div>
                                <div class="column form-column">
                                    <h2 class="title is-2 has-text-centered">Booking Form</h2>
                                    <form id="booking-form" @submit.prevent="processForm">
                                    <!-- name -->
                                    <div class="field">
                                        <label class="label">Nama</label>
                                        <div class="control">
                                            <input type="text" class="input" name="name" placeholder="e.g. John Doe" v-model="form.name">
                                            <!-- <div class="field-error-message" v-if="!$v.form.name.required">Nama tidak boleh kosong</div> -->
                                        </div>
                                    </div>
                                    <div class="columns" style="margin-bottom:0px">
                                        <div class="column">
                                            <!-- email -->
                                            <div class="field">
                                                <label class="label">Alamat Email</label>
                                                <div class="control">
                                                    <input type="email" class="input" name="email" placeholder="e.g. john@mail.com" v-model="form.email">
                                                    <!-- <div class="field-error-message" v-if="!$v.form.email.required">Alamat email tidak boleh kosong</div> -->
                                                    <!-- <div class="field-error-message" v-if="!$v.form.email.email">Format alamat email salah</div> -->
                                                </div>
                                            </div>
                                        </div>
                                        <div class="column">
                                            <!-- phone -->
                                            <div class="field">
                                                <label class="label">Nomor Telepon</label>
                                                <div class="control">
                                                    <input type="tel" class="input" name="phone" placeholder="e.g. 08123456789" v-model="form.phone">
                                                    <!-- <div class="field-error-message" v-if="!$v.form.phone.required">Nomor telepon tidak boleh kosong</div> -->
                                                    <!-- <div class="field-error-message" v-if="(!$v.form.phone.minLength) || (!$v.form.phone.maxLength)">Panjang nomor telepon harus antara 8 sampai 15 digit</div> -->
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                    <!-- address -->
                                    <div class="field">
                                        <label class="label">Alamat</label>
                                        <div class="control">
                                            <textarea class="textarea" name="address" rows="3" v-model="form.address"></textarea>
                                            <!-- <div class="field-error-message" v-if="!$v.form.address.required">Alamat tidak boleh kosong</div> -->
                                        </div>
                                    </div>

                                    <!-- service booking -->
                                    <div class="field">
                                        <label class="label">Pilihan Paket</label>
                                        <div class="control">
                                            <div class="select is-fullwidth">
                                            <select v-model.number="form.package_id" @change="getPrice">
                                                <option selected value=0>Pilih Paket...</option>
                                                <option value=1>Murah Meriah di Rumah</option>
                                                <option value=2>Steam Biasa</option>
                                                <option value=3>Steam + Interior Cleaning</option>
                                                <option value=4>Steam + Ultimate Wax</option>
                                                <option value=5>Ultimate Steam + Wax Machine</option>
                                                <option value=6>Ultimate Steam + Wax Manual + Interior Cleaning</option>
                                                <option value=7>Ultimate Steam + Wax Machine + Interior Cleaning</option>
                                            </select>
                                            </div>
                                            <!-- <div class="field-error-message" v-if="!$v.form.address.minValue">Harus memilih salah satu paket</div> -->
                                        </div>
                                    </div>
                                    <div class="field">
                                        <label class="label">Tanggal &amp; Jam</label>
                                        <div class="columns" style="margin-bottom: 0px;">
                                            <div class="column" style="padding-bottom: 0px;">
                                                    <div class="control">
                                                        <datetime
                                                            type="date"
                                                            v-model="date"
                                                            value-zone="Asia/Jakarta"
                                                            :min-datetime="new Date(minDate.setDate(today.getDate() + 1)).toISOString()"
                                                            :max-datetime="new Date(maxDate.setDate(minDate.getDate() + 7)).toISOString()">
                                                        </datetime>
                                                    </div>
                                                </div>
                                            <div class="column" style="padding-bottom: 0px;">
                                                <div class="control">
                                                    <div class="select is-fullwidth">
                                                    <select v-model="form.datetime">
                                                        <option selected :value="null">Jam Tersedia...</option>
                                                        <option
                                                            v-for="(data, index) in availableTime"
                                                            :key="index"
                                                            v-if="new Date(data).getDate() === new Date(date).getDate()"
                                                            :value="data">
                                                            {{ formatTimeString(new Date(data)) }}
                                                        </option>
                                                    </select>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                        <!-- <div class="field-error-message" v-if="!$v.form.datetime.required">Harus memilih jadwal yang disediakan</div> -->
                                    </div>
                                    <div class="field">
                                        <label class="label">Tambahan</label>
                                        <div class="columns">
                                            <div class="column">
                                                <label class="checkbox">
                                                    <input type="checkbox" v-model="form.extra_water" @change="getPrice">&nbsp;Air
                                                </label>
                                            </div>
                                            <div class="column">
                                                <label class="checkbox">
                                                    <input type="checkbox" v-model="form.extra_electricity" @change="getPrice">&nbsp;Listrik
                                                </label>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="field">
                                        <label class="label">Harga Total</label>
                                        <div v-if="isPriceLoading" class="button is-loading is-link"><h5 class="title is-5 price-loading">Rp {{ formatPrice(0) }}</h5></div>
                                        <div v-else class="button is-hovered is-link"><h5 class="title is-5 price">Rp {{ formatPrice(form.price) }}</h5></div>
                                    </div>
                                    <div v-if="submitSuccess" class="notification is-success">
                                        <strong>Booking Sukses!</strong><br>
                                        Booking Anda telah diproses. Kami akan mengirimkan detail booking Anda melalui email.
                                        Terimakasih telah mempercayakan perawatan mobil Anda kepada kami.<br>
                                        <router-link to="/Home"><a>Klik di sini</a></router-link> untuk kembali ke halaman utama.
                                    </div>
                                    <div v-if="submitError" class="notification is-danger">
                                        <strong>Booking Gagal!</strong><br>
                                        Kami menemui suatu masalah ketika memproses booking Anda. Mohon coba lagi. Tolong periksa kembali isian form Anda. Mohon maaf atas ketidaknyamanan yang ditimbulkan.<br>
                                        <router-link to="/Home"><a>Klik di sini</a></router-link> untuk kembali ke halaman utama.
                                    </div>
                                    <!-- submit button -->
                                    <div class="field has-text-right">
                                        <span v-if="isSubmitting" class="button is-loading is-danger">Submit</span>
                                        <button v-else type="submit" class="button is-danger">Submit</button>
                                    </div>
                                    </form>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
const axios = require('axios')
import { required, email, numeric, minValue, minLength, maxLength } from 'vuelidate/lib/validators'

export default {
  name: 'BookingForm',
  created: async function () {
        let url = 'https://formula-variasi.herokuapp.com/carcare/api/availabledatetime/'
        await axios.get(url)
        .then((response) => {
            console.log(response)
        })
        .catch((error) => {
            console.log(error)
        })
  },
  data() {
    return {
        isPriceLoading: false,
        today: new Date(),
        minDate: new Date(),
        maxDate: new Date(),
        availableTime: [],
        date: null,
        isSubmitting: false,
        submitSuccess: false,
        submitError: false,
        form: {
            name: null,
            email: null,
            phone: null,
            address: null,
            package_id: 0,
            datetime: null,
            extra_water: false,
            extra_electricity: false,
            price: null
        }
    }
  },
  validations: {
    form: {
        name: {
            required
        },
        email: {
            required,
            email
        },
        phone: {
            required,
            numeric,
            minLength: minLength(8),
            maxLength: maxLength(15)
        },
        address: {
            required
        },
        package_id: {
            required,
            minValue: minValue(0)
        },
        datetime: {
            required
        }
    }
  },
  methods: {
    processForm: async function() {
        // this.$v.form.$touch()
        // if (this.$v.form.$invalid) {
        //     console.log('invalid ber')
        // }
        // else {
            this.isSubmitting = true
            await axios.post('https://formula-variasi.herokuapp.com/carcare/api/booking/', {
                name: this.form.name,
                email: this.form.email,
                phone: this.form.phone,
                address: this.form.address,
                package_id: this.form.package_id,
                datetime: this.form.datetime,
                datetime: this.form.datetime,
                extra_water: this.form.extra_water,
                extra_electricity: this.form.extra_electricity,
                price: this.form.price
            })
            .then((response) => {
                console.log(response)
                this.submitSuccess = true
                this.submitError = false
            })
            .catch((error) => {
                console.log(error)
                this.submitSuccess = false
                this.submitError = true
            });

            this.isSubmitting = false
        // }
    },
    formatPrice(value) {
        let val = (value/1).toFixed(2).replace('.', ',')
        return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".")
    },
    getPrice: async function () {
        this.isPriceLoading = true
        console.log("this.form.price = " + this.form.price)
        console.log("this.form.package_id = " + this.form.package_id)
        if (this.form.package_id === 0 || !this.form.package_id) {
            this.form.price = 0
        }
        else {
            let url = 'https://formula-variasi.herokuapp.com/carcare/api/packages?id='
            await axios.get(url + this.form.package_id)
            .then((response) => {
                // handle success
                console.log(response)
                this.form.price = parseInt(response.data[0].package_price, 10)
            })
            .catch((error) => {
                // handle error
                console.log(error)
            })

            if (this.form.extra_water) {
                this.form.price += 5000
            }

            if (this.form.extra_electricity) {
                this.form.price += 15000
            }
        }
        this.isPriceLoading = false
        console.log(this.form.price)
    },
    getTime: async function () {
        let url = 'https://formula-variasi.herokuapp.com/carcare/api/availabledatetime/'
        await axios.get(url)
        .then((response) => {
            console.log(response)
            this.availableTime = response.data
        })
        .catch((error) => {
            console.log(error)
        })
        console.log(this.availableTime)
    },
    formatTwoDigitString(n) {
        return n > 9 ? "" + n : "0" + n;
    },
    formatTimeString(date) {
        return this.formatTwoDigitString(date.getHours()) + ":" + this.formatTwoDigitString(date.getMinutes())
    }
  },
  watch: {
    date: function () {
        this.getTime()
        console.log(this.date)
        console.log(new Date(this.date).getHours())
        if (new Date(this.availableTime[4]).getDate() === new Date(this.date).getDate()) {
            console.log('sama')
        }
        else {
            console.log('beda')
        }
        console.log(new Date(this.availableTime[4]))
        console.log(new Date(this.date))
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.full-form {
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}

.decoration-column {
    background-image: url('../assets/black01.jpg');
    background-size: cover;
    background-position: left;
}

.form-column {
    background-color: rgba(43, 62, 79, 1);
}

.price {
    margin-bottom: 0px;
    color: white;
}

.price-loading {
    color: rgba(0, 0, 0, 0)
}
</style>
