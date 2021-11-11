<script lang="ts">
	var params = {
		appName: "graphing",
		scaleContainerClass: "main",
		showToolBar: true,
		showAlgebraInput: true,
		showMenuBar: true,
	};

	params.appletOnLoad = function (api) {
		function printConstructionState() {
			let pointsList: Array<[number, number]> = [];

			let objNumber = api.getObjectNumber();

			// collect co-ordinates of all 'point' objects
			for (var i = 0; i < objNumber; i++) {
				let strName: string = api.getObjectName(i);
				let strType: string = api.getObjectType(strName);

				// if the object is not a point, don't bother
				if (strType != "point") {
					continue;
				}

				// otherwise record co-ordinates
				pointsList.push([
					api.getXcoord(strName),
					api.getYcoord(strName),
				]);
			}

			let myBigPolynomial: string = "";
			for (let pointGoal of pointsList) {
				// create a polynomial that caters to a particular node
				let myPolynomial: string = `${pointGoal[1]}`;
				for (let pointZero of pointsList) {
					// make the polynomial equal to zero at every other node
					// the second term in the product is necessary for scaling the polynomial for the chosen node
					if (pointZero == pointGoal) {
						continue;
					}
					myPolynomial += `*(x-${pointZero[0]})*(${pointGoal[0]}-${pointZero[0]})^-1`;
				}
				// add the finished polynomial to the one to be plotted
				myBigPolynomial += `+(${myPolynomial})`;
			}

			// unregister listener temporarily to prevent function from calling itself
			api.unregisterUpdateListener(printConstructionState);
			// update function f with final polynomial
			api.evalCommand(`ParseToFunction(f, "${myBigPolynomial}")`);
			api.registerUpdateListener(printConstructionState);
		}
		// draw a function (placeholder) and rename it so we can reference f later
		api.renameObject(api.evalCommandGetLabels("x"), "f");

		// react to new points created or old points moved
		api.registerAddListener(printConstructionState);
		api.registerUpdateListener(printConstructionState);
	};

	// initialise applet
	var applet = new GGBApplet(params, true);

	window.addEventListener("load", function () {
		applet.inject("ggb-element");
	});
</script>

<div class="main" id="ggb-element" />

<style>
	.main {
		height: 100%;
		width: 100%;
	}
</style>
