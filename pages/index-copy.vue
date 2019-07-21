<template lang="pug">
v-container(grid-list-md text-xs-center)
	v-layout(row wrap)
		v-flex(xs12)
			v-card.pt-4
				.uniform.ma-2
					img.image(
						:src="image"
					)
				.uniform.ma-2
					img.image(
						:src="stylePreview"
					)
				.uniform.ma-2
					img#image.image(
						:src="imageSrc"
					)
				v-card-actions.mx-3
					v-card-text
						v-dialog(
							width="500"
							v-model="selectSample"
						)
							template(v-slot:activator="{ on }")
								v-btn.mx-1(
									dark
									v-on="on"
									@click="startCapture"
								) Select Image
							v-flex(xs12)
								v-card
									v-container(grid-list-sm fluid)
										v-layout(row wrap)
											v-flex(
												v-for="sample in samples"
												:key="sample"
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
															:src="sample"
															aspect-ratio="1"
															class="grey lighten-2"
															@click="setSample(sample)"
														)
											
						v-dialog(
							width="500"
							v-model="takePicture"
						)
							template(v-slot:activator="{ on }")
								v-btn.mx-1(
									dark
									v-on="on"
									@click="startCapture"
								) Take Picture
							v-flex(xs12)
								v-card.pa-4
									canvas#canvas(
										width="320" height="320"
									)
									.video-container
										video#video(
											width="320" height="320"
											autoplay=true
										)
									v-card-actions
										v-spacer
										v-btn(
											dark
											@click="capture"
										) Take Picture
										v-spacer
					v-spacer
					v-card-text
						v-dialog(
							width="500"
							v-model="selectStyle"
						)
							template(v-slot:activator="{ on }")
								v-btn.mx-1(
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
															class="grey lighten-2"
															@click="setStyle(style)"
														)
					v-card-text
						v-btn.mx-1(
							color="primary"
							@click="styletransfer"
						) Transfer Style
						v-spacer
</template>

<script>
import ml5 from 'ml5'

const constraints =  {
	"video": {
		width: {
			exact: 320
		}
	}
}

export default {
	data() {
		return {
			style: undefined,
			image: '/samples/puppy.jpg',
			imageSrc: '/samples/puppy.jpg',
			MediaStream: window.MediaStream,
			selectStyle: false,
			selectSample: false,
			takePicture: false,
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
			samples: [
				'/samples/puppy.jpg',
				'/samples/arch.jpg',
				'/samples/vermeer.jpg'
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
		this.style = ml5.styleTransfer(this.styleModel, () => {
			console.log('model loaded')
		})
		this.cropImage('/samples/vermeer.jpg')
	},
	methods: {
		setStyle(style) {
			this.selectStyle = false
			this.model = style
			this.imageSrc = this.image
			
			this.style = ml5.styleTransfer(this.styleModel, () => {
				console.log('model loaded')
			})
		},
		setSample(sample) {
			this.selectSample = false
			this.image = sample
			this.imageSrc = sample
		},
		async styletransfer() {
			let image = document.getElementById('image')
	
			await this.style.transfer(image, (err, result) => {
				this.imageSrc = result.src
			})
		},
		startCapture() {
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
			this.takePicture = false
		},
		capture() {
			navigator.mediaDevices.getUserMedia({video: true})
  				.then(this.gotMedia)
				.catch(error => console.error('getUserMedia() error:', error))
			this.stopCapture()
		},
		gotMedia(mediaStream) {
			let imageHeight, imageWidth

			const mediaStreamTrack = mediaStream.getVideoTracks()[0]
			const imageCapture = new ImageCapture(mediaStreamTrack)

			imageCapture.takePhoto()
				.then(blob => {
					this.imageSrc = URL.createObjectURL(blob)

					// const canvas = document.createElement('canvas')
					let canvas = document.getElementById('canvas')
					canvas.width = 320
					canvas.height = 320
					const ctx = canvas.getContext('2d')

					const img = new Image()

					imageCapture.getPhotoSettings()
						.then(({ imageHeight, imageWidth }) => {
							let ratio = imageWidth / imageHeight
							let width = imageHeight * ratio
							let height = imageWidth / ratio
							let xOffset = 0
							let yOffset = 0

							if (width < canvas.width) {
								let zoom = canvas.width / width
								width = zoom * width
								height = zoom * height
								yOffset = -(height - canvas.height) / 2
							}

							if (height < canvas.height) {
								let zoom = canvas.height / height
								width = zoom * width
								height = zoom * height
								xOffset = -(width - canvas.width) / 2
							}
							
							img.onload = () => {
								ctx.drawImage(img, xOffset, yOffset, width / 2, height / 2)
								let canvasImg = canvas.toDataURL("image/jpeg")
								this.imageSrc = this.image = canvasImg
							}


							img.src = this.imageSrc
							this.samples.push(this.imageSrc)
						})
						.catch(err => console.error(err))
				})
				.catch(error => console.error('takePhoto() error:', error))
		},
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

.image {
	position: absolute;
	height: 100%;
	width: auto;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
}

.video-container, canvas {
	height: 320px;
	width: 320px;
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
    top: 0;
    left: 50%;
    transform: translateX(-50%);
}

#canvas {
	display: none;
}
</style>
