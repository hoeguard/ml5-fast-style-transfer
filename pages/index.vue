<template lang="pug">
v-container(grid-list-md text-xs-center)
	v-layout(row wrap)
		v-flex(xs12)
			v-card
				v-layout(row wrap)
					v-flex.pa-0(xs4)
						v-card.px-2.py-4(flat v-if="sample")
							v-tooltip(top :disabled="!sample.attribution")
								template(v-slot:activator="{ on }")
									v-img(
										v-if="sample.src"
										v-on="on"
										width="100%"
										aspect-ratio="1"
										:src="sample.src"
									)
								small(v-if="sample.attribution") {{ sample.attribution }}
							v-card-actions.pt-4.pb-0
								v-spacer
								v-dialog(
									width="500"
									v-model="sampleDialog"
								)
									template(v-slot:activator="{ on }")
										v-btn.mx-1(
											:disabled="loading"
											dark
											v-on="on"
										) Select Image
									v-flex(xs12)
										v-card
											v-container(grid-list-sm fluid)
												v-layout(row wrap)
													v-flex(
														v-for="sample in samples"
														:key="sample.name"
														xs4
													)
														v-hover
															v-card.ma-1(
																tile
																slot-scope="{ hover }"
																:class="`elevation-${hover ? 12 : 2}`"
																:style="{ cursor: 'pointer'}"
																@click="setSample(sample)"
															)
																v-img(
																	:src="sample.src"
																	aspect-ratio="1"
																)
												v-layout(row wrap)
													v-tooltip(bottom)
														template(v-slot:activator="{ on }")
															v-flex(xs4 v-on="on")
																v-hover
																	v-card.ma-1(
																		tile
																		aspect-ratio="1"
																		slot-scope="{ hover }"
																		:class="`elevation-${hover ? 12 : 2}`"
																		:style="{ cursor: 'pointer'}"
																		@click="startCapture"
																	)
																		v-card.grey.darken-4
																			v-responsive
																				v-img.ma-5(
																					contain
																					src="/icons/camera-solid.svg"
																				)
														span Take Photo
																		
													v-tooltip(bottom)
														template(v-slot:activator="{ on }")
															v-flex(xs4 v-on="on")
																v-hover
																	v-card.ma-1(
																		tile
																		aspect-ratio="1"
																		slot-scope="{ hover }"
																		:class="`elevation-${hover ? 12 : 2}`"
																		:style="{ cursor: 'pointer'}"
																		@click="$refs.imageUpload.click()"
																	)
																		v-card.grey.darken-4
																			v-responsive
																				v-img.ma-5(
																					contain
																					src="/icons/image-solid.svg"
																				)
														span Upload Photo
													
								v-dialog(
									width="500"
									v-model="photoDialog"
								)
									template(v-slot:activator="{ on }")
									v-flex(xs12)
										v-card.pa-4
											.video-container
												video#video(
													width="800" height="800"
													autoplay=true
												)
											v-card-actions.pt-4.pb-0
												v-spacer
												v-btn(
													dark
													@click="capture"
												) Take Photo
												v-spacer

								v-spacer
					
					v-flex.pa-0(xs4)
						v-card.px-2.py-4(flat)
							v-img(
								width="100%"
								aspect-ratio="1"
								:src="stylePreview"
							)
							v-card-actions.pt-4.pb-0
								v-spacer
								v-dialog(
									width="500"
									v-model="styleDialog"
								)
									template(v-slot:activator="{ on }")
										v-btn.mx-1(
											:disabled="loading"
											dark
											v-on="on"
										) Select Style

									v-flex(xs12)
										v-card
											v-container(grid-list-sm fluid)
												v-layout(row wrap)
													v-flex(
														v-for="style in styles"
														:key="style"
														xs4
													)
														v-hover
															v-card.ma-1(
																tile
																slot-scope="{ hover }"
																:class="`elevation-${hover ? 12 : 2}`"
																:style="{ cursor: 'pointer'}"
															)
																v-img(
																	:src="`/styles/${style}.jpg`"
																	aspect-ratio="1"
																	@click="setStyle(style)"
																)
								v-spacer

					v-flex.pa-0(xs4)
						v-card.px-2.py-4(flat)
							.tf-image
								img#image(
									:src="imageSrc"
							)
							v-card-actions.pt-4.pb-0
								v-spacer
								v-btn.mx-1(
									:disabled="loading"
									color="primary"
									@click="styletransfer"
								) Transfer Style
								v-spacer

	input(
		v-show="false"
		ref="imageUpload"
		type="file"
		accept=".jpg, .jpeg"
		@change="imageUpload"
	)
	
	v-snackbar(
		v-model="snackbar"
		bottom
		:timeout="4000"
	) 
		v-spacer
		span {{ log }}
		v-spacer
</template>

<script>
import ml5 from 'ml5'

const constraints =  {
	"video": {
		width: {
			exact: 1200
		}
	}
}

export default {
	data() {
		return {
			style: undefined,
			image: '/samples/arch.jpg',
			imageSrc: '/samples/arch.jpg',
			pixels: 640,
			imageReader: {},
			MediaStream: {},
			styleDialog: false,
			sampleDialog: false,
			photoDialog: false,
			snackbar: false,
			log: '',
			loading: false,
			model: 'udnie',
			styles: [
				'fuchun',
				'la_muse',
				'rain_princess',
				'scream',
				'signac',
				'udnie',
				'wave',
				'wreck'
			],
			sample: undefined,
			samples: [
				{
					src: "/samples/arch.jpg",
					attribution: "St_Louis_night_expblend.jpg: Daniel Schwenderivative work: ←fetchcomms [CC BY-SA 2.5 (https://creativecommons.org/licenses/by-sa/2.5)]"
				},
				{
					src: "/samples/cage.jpg",
					attribution: "nicolas genin [CC BY-SA 2.0 (https://creativecommons.org/licenses/by-sa/2.0)]"
				},
				{
					src: "/samples/vangogh.jpg"
				},
				{
					src: "/samples/vermeer.jpg"
				},
				{
					src: "/samples/stlouis.jpg",
					attribution: "Colin Faulkingham [Public domain]"
				}
			]
		}
	},
	computed: {
		stylePreview() {
			return `/styles/${this.model}.jpg`
		},
		styleModel() {
			return `/models/${this.model}`
		}
	},
	mounted() {
		this.MediaStream = window.MediaStream
		this.imageReader = new imageReader()
		this.sample = this.samples[0]
		this.setStyle(this.model)
	},
	methods: {
		setStyle(style) {
			this.loading = true
			this.snackbar = false
			this.styleDialog = false
			this.model = style
			this.imageSrc = this.sample.src

			const start = performance.now()
			
			this.style = ml5.styleTransfer(this.styleModel, () => {
				const end = performance.now()
				this.log = `Model '${style}' loaded in ${Math.round(end - start)}ms`
				this.snackbar = true
				this.loading = false
			})
		},
		setSample(sample) {
			this.sampleDialog = false
			this.sample = sample
			this.imageSrc = sample.src
		},
		async styletransfer() {
			this.loading = true
			this.snackbar = false
			const image = document.getElementById('image')
			const start = performance.now()

			await this.style.transfer(image, (err, result) => {
				if (err) {
					console.log(err)
					return
				}
				const end = performance.now()
				this.log = `Style transferred in ${Math.round(end - start)}ms`
				this.snackbar = true
				this.loading = false
				this.imageSrc = result.src
			})
		},
		startCapture() {
			this.styleDialog = this.sampleDialog = false
			this.photoDialog = true
			navigator.mediaDevices.getUserMedia(constraints)
				.then(this.streamCapture)
				.catch(e => console.log(e))
		},
		streamCapture(MediaStream) {
			this.MediaStream = MediaStream
			document.getElementById('video').srcObject = this.MediaStream
		},
		stopCapture() {
			this.MediaStream.getVideoTracks()[0].stop()
			this.photoDialog = false
		},
		capture() {
			navigator.mediaDevices.getUserMedia({video: true})
  				.then(this.retrieveMedia)
				.catch(error => console.error('getUserMedia() error:', error))
			this.stopCapture()
		},
		retrieveMedia(mediaStream) {
			let imageHeight, imageWidth

			const mediaStreamTrack = mediaStream.getVideoTracks()[0]
			const imageCapture = new ImageCapture(mediaStreamTrack)

			imageCapture.takePhoto()
				.then(async blob => {
					this.setImage(blob)
				})
				.catch(error => console.error('takePhoto() error:', error))
		},
		async imageUpload(e) {
			this.styleDialog = this.sampleDialog = this.photoDialog = false
			if (e.srcElement.files[0].type !== "image/jpeg") {
				console.log('File must be of type image/jpeg')
				return
			}
			let file = e.srcElement.files[0]
			
			this.setImage(file)
		},
		async setImage(file) {
			this.loading = true
			await this.imageReader.read(file)
			const dataURL = await this.imageReader.resize(this.pixels, this.pixels)

			this.loading = false
			this.samples.push({ src: dataURL })
			this.sample = this.samples[this.samples.length - 1]
			this.imageSrc = dataURL
		}
	}
}

// https://codepen.io/ryanhogard/pen/rXVmzZ
function imageReader() {
	this.dataURL = undefined
	
	this.read = file => {
		const reader = new FileReader()
		
		return new Promise((resolve, reject) => {
			reader.onerror = error => {
				reader.abort()
				reject(new DOMException(error))
			}
			reader.onload = () => {
				this.dataURL = reader.result
				resolve(reader.result)
			}
			reader.readAsDataURL(file)
		})
	}
	
	this.resize = (width, height, dataURL = undefined) => {
		const img = new Image()
		
		return new Promise((resolve, reject) => {
			const canvas = document.createElement('canvas')
			canvas.width = width
			canvas.height = height
			const ctx = canvas.getContext('2d')
			
			img.onerror = error => {
				reject(new DOMException(error))
			}
			img.onload = () => {
				const ratio = img.width / img.height
				let width = img.width
				let height  = img.height
				let scale = 1
				let x = 0
				let y = 0
				
				// upscale || downscale
				if (
					width < canvas.width || height < canvas.height 
					 || width > canvas.width && height > canvas.height
				) {
					if (width - canvas.width < (height - canvas.height) * ratio)  {
						scale = canvas.width / width
					} else {
						scale = canvas.height / height
					}
					
					// scale
					width = scale * width
					height = scale * height
					
					// center
					y = -(height - canvas.height) / 2
					x = -(width - canvas.width) / 2
				}
				
				ctx.drawImage(img, x, y, width, height)
				resolve(canvas.toDataURL("image/jpeg"))
			}
			
			img.src = dataURL || this.dataURL
		})
	}
}
</script>

<style lang="styl">
.uniform {
	width: 30%;
	padding-top: 30%;
	position: relative;
	display: inline-block;
	overflow: hidden;
}

.tf-image {
	position: relative;
	width: 100%;
	height: 0;
	padding-top: 100%;
	overflow: hidden;
}

#image {
	position: absolute;
	height: 100%;
	width: auto;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
}

.video-container, canvas {
	height: 400px;
	width: 400px;
	position: relative;
	overflow: hidden;
	margin: 0 auto;
}

#video {
	position: absolute;
	min-width: 100%;
	min-height: 100%;
	width: auto;
	height: auto;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%) scale(0.6);
}
</style>
