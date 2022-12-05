/* eslint-disable */

export default () => {
	let overlay = '<div class="waiting-overlay">&nbsp;</div>',
		globalStates = {}, // Gets populated on the initial feed pull for states
		restURL = "https://stthomas.force.com/applicantportal/services/apexrest/";

	$(document).ready(() => {
		// ====================
		//  On Load
		// ====================

		pullFeed(
			"state",
			{
				element: $("#states"),
			},
			"ustadmissionsrest.json"
		);

		$("#describeYou").change(function () {
			const desc = $(this).val();
			const elOpts = [$("#statesWrap"), $("#schoolsWrap"), $("#transferWrap")];
			$.each(elOpts, function () {
				$(this).hide();
				$(this).find("select option[value='']").attr("selected", "selected");
			});
			switch (desc) {
				case "hs":
					elOpts[0].show();
					break;
				case "trsf":
					elOpts[2].show();
					break;
				case "intl":
					//SchoolId, state, fullBio, name, nation, territory, feedType, role
					getCounselor("", "", true, "", "", "", "intl", "");
					break;
				case "expat":
					getCounselor("", "", true, "", "Italy", "", "counselor", "");
					break;
				case "vet":
					getCounselor("", "", true, "Nathan Theunissen", "", "", "counselor", "U24");
					break;
			}
		});

		$("#tranferFrom").change(function () {
			if ($(this).val() == "SCHTYPE-INTLC") {
				getCounselor("", "", true, "", "", "", "intl");
			} else {
				getCounselor("", "", true, "", "", $(this).val(), "counselor");
			}
		});

		$("#states").change(function () {
			$("#schoolsWrap").hide();
			$("#counselorView").html("");
			let state = $(this).val().toLowerCase();
			if (state == "mn" || state == "wi") {
				$("#schoolsWrap").show().append(overlay);
				pullFeed(
					"highSchool",
					{
						element: $("#schools"),
						data: {
							state: state,
						},
					},
					"ustadmissionsrest.json"
				);
				$("#schools").change(function () {
					//SchoolId, state, fullBio, name, nation, territory, feedType, role
					getCounselor($(this).val(), "", true, "", "", "", "counselor", "");
				});
			} else {
				//SSchoolId, state, fullBio, name, nation, territory, feedType, role
				getCounselor("", state, true, "", "", "", "counselor", "");
			}
		});

		$("head").append(`
        <style>
            h3, p, table {
                font-family: Avenir;
            }
            p, table {
                font-weight: 400;
            }
            
            h3 {
                font-size: 1.5rem;
                color: #3c045b;
                font-weight: 700;
            }
        </style>
    `);
	});

	function getCounselor(SchoolId, state, fullBio, name, nation, territory, feedType, role) {
		$("#counselorView").html(overlay);
		pullFeed(
			"counselor",
			{
				element: $("#counselorView"),
				data: {
					type: feedType, // 'counselor', 'bio',
					name: name,
					school: SchoolId,
					state: state,
					nation: nation,
					territory: territory,
					fullbio: fullBio,
					role: role,
				},
				callback() {},
			},
			"admissioncounselor.json"
		);
	}

	function counselorDisplayList($element, data) {}

	function counselorDisplayFull($element, data) {
		$element.html("");
		$.each(data, (key, val) => {
			var cList = $("<div/>").addClass("grid-x grid-padding-x align-center");

			var picCol = $("<div/>").addClass("locate-img column small-12 medium-6 large-6");
			picCol.append($("<img/>").attr("src", val.largePhoto));

			//Contact Information
			var infoCol = $("<div/>").addClass("column small-12 medium-5 large-5");
			var contactBlock = $("<table/>");
			if (val.phone) {
				var telFormat = val.phone.split("ext");
				telFormat = telFormat[0];
				telFormat = telFormat.replace(/\D/g, "");
				contactBlock.append(
					$("<tr/>")
						.append($("<td/>").html("Phone"))
						.append($("<td/>").html('<a href="tel:' + telFormat + '">' + val.phone + "</a>"))
				);
			}
			if (val.email) {
				contactBlock.append(
					$("<tr/>")
						.append($("<td/>").html("Email"))
						.append($("<td/>").html('<a href="mailto:' + val.email + '">' + val.email + "</a>"))
				);
			}
			if (val.college) {
				contactBlock.append($("<tr/>").append($("<td/>").html("College")).append($("<td/>").html(val.college)));
			}
			if (val.major) {
				contactBlock.append($("<tr/>").append($("<td/>").html("Major")).append($("<td/>").html(val.major)));
			}
			if (val.hometown) {
				contactBlock.append($("<tr/>").append($("<td/>").html("Hometown")).append($("<td/>").html(val.hometown)));
			}
			if (val.address) {
				contactBlock.append($("<tr/>").append($("<td/>").html("Address")).append($("<td/>").html(val.address)));
			}

			infoCol.append($("<h3/>").html(val.name));
			infoCol.append($("<p/>").html(val.title));
			infoCol.append(contactBlock);
			cList.append(picCol);
			cList.append(infoCol);
			$element.append(cList);

			//about block
			var aboutCol = $("<div/>").addClass("grid-padding-x align-center");
			if (val.about) {
				aboutCol.append($("<h3/>").html("A little about me"));
				aboutCol.append($("<p/>").html(val.about));
			}
			if (val.USTLove) {
				aboutCol.append($("<h3/>").html("Why I choose St. Thomas"));
				aboutCol.append($("<p/>").html(val.USTLove));
			}
			if (val.advice) {
				aboutCol.append($("<h3/>").html("A word of advice when choosing a college"));
				aboutCol.append($("<p/>").html(val.advice));
			}

			$element.append($("<div/>").addClass("column").append(aboutCol));
		});
	}

	function pullFeed(feedType, options, url) {
		let defaultText = "Select...";
		let requestData = !(typeof options === "undefined") ? options.data : false; // Get the data to send in the AJAX request
		let $element = !(typeof options === "undefined") ? options.element : false; // Get the element to modify with the data
		let elementTag = !(typeof $element === "undefined") && !(typeof $element.prop("tagName") === "undefined") ? $element.prop("tagName").toLowerCase() : false; // Get that element's type
		let callback = !(typeof options === "undefined") ? options.callback : false; // Get the callback to run after everything is complete

		// All supported feeds
		let feeds = {
			state() {
				/*
              Options: {}
              */
				var states = {
					AA: "U.S. Armed Forces â€“ Americas HIDE",
					AE: "U.S. Armed Forces â€“ Europe HIDE",
					AL: "Alabama",
					AK: "Alaska",
					AP: "U.S. Armed Forces â€“ Pacific HIDE",
					AZ: "Arizona",
					AR: "Arkansas",
					CA: "California",
					CO: "Colorado",
					CT: "Connecticut",
					DE: "Delaware",
					FL: "Florida",
					FM: "Micronesia HIDE",
					GA: "Georgia",
					HI: "Hawaii",
					ID: "Idaho",
					IL: "Illinois",
					IN: "Indiana",
					IA: "Iowa",
					KS: "Kansas",
					KY: "Kentucky",
					LA: "Louisiana",
					ME: "Maine",
					MD: "Maryland",
					MA: "Massachusetts",
					MH: "Marshall Islands HIDE",
					MI: "Michigan",
					MN: "Minnesota",
					MP: "Northern Mariana Islands HIDE",
					MS: "Mississippi",
					MO: "Missouri",
					MT: "Montana",
					NE: "Nebraska",
					NV: "Nevada",
					NH: "New Hampshire",
					NJ: "New Jersey",
					NM: "New Mexico",
					NY: "New York",
					NC: "North Carolina",
					ND: "North Dakota",
					OH: "Ohio",
					OK: "Oklahoma",
					OR: "Oregon",
					PA: "Pennsylvania",
					PW: "Palau HIDE",
					RI: "Rhode Island",
					SC: "South Carolina",
					SD: "South Dakota",
					TN: "Tennessee",
					TX: "Texas",
					UT: "Utah",
					UM: "U.S. Minor Outlying Islands HIDE",
					VT: "Vermont",
					VA: "Virginia",
					WA: "Washington",
					WV: "West Virginia",
					WI: "Wisconsin",
					WY: "Wyoming",
					AB: "Alberta HIDE",
					AS: "American Samoa",
					BC: "British Columbia HIDE",
					DC: "District of Columbia",
					GU: "Guam",
					MB: "Manitoba HIDE",
					NB: "New Brunswick HIDE",
					NL: "Newfoundland and Labrador HIDE",
					NS: "Nova Scotia HIDE",
					NT: "Northwest Territories HIDE",
					NU: "Nunavut HIDE",
					ON: "Ontario HIDE",
					PE: "Prince Edward Island HIDE",
					PR: "Puerto Rico HIDE",
					QC: "Quebec HIDE",
					SK: "Saskatchewan HIDE",
					VI: "Virgin Islands",
					YT: "Yukon HIDE",
				};
				url += "?type=states";
				defaultText = "Select a state...";

				makeRequest(url, requestData, function (data) {
					var fixedOutput = {};
					$.each(data, function (key, val) {
						if (!states[key].endsWith("HIDE")) {
							fixedOutput[key] = states[key] || val;
						}
					});
					if (elementTag) tags[elementTag](fixedOutput);
					globalStates = fixedOutput;
				});
			},
			highSchool() {
				/*
              Options: {
                  state: required
                  city: optional
                  filter: optional
              }
              */
				url += "?type=highschool";
				defaultText = "Select a school...";
				makeRequest(url, requestData, function (data) {
					if (elementTag) tags[elementTag](data);
				});
			},
			counselor() {
				/*
              Options: {
                  state: required
                  city: optional
                  filter: optional
              }
              */
				makeRequest(url, requestData, function (data) {
					if (requestData.fullbio) {
						counselorDisplayFull($element, data);
					} else {
						counselorDisplayList($element, data);
					}
				});
			},
		};

		// All supported tags
		var tags = {
			select(data) {
				var inData = sortMapByValue(data, false);
				$element.html("");
				$("<option/>", {
					disabled: "disabled",
					selected: "selected",
					default: "",
					text: defaultText,
				}).appendTo($element);
				for (var i = 0; i < inData.length; i++) {
					$("<option/>", {
						value: inData[i][0],
						text: inData[i][1],
					}).appendTo($element);
				}
				$("#schoolsWrap").find(".waiting-overlay").remove();
				if (callback) callback();
			},
		};

		// Run the specified feed
		feeds[feedType]();

		// Get the feed with the specified data and run a callback when complete
		function makeRequest(url, data, callback) {
			$.ajax({
				url: restURL + url,
				dataType: "jsonp",
				data,
			}).done((output) => {
				callback(output);
			});
		}
	}

	function getUrlParameter(sParam) {
		let sPageURL = decodeURIComponent(window.location.search.substring(1)),
			sURLVariables = sPageURL.split("&"),
			sParameterName;

		for (let i = 0; i < sURLVariables.length; i++) {
			sParameterName = sURLVariables[i].split("=");
			if (sParameterName[0] === sParam) return sParameterName[1] === undefined ? true : sParameterName[1];
		}
	}

	function sortMapByValue(obj, isNumericSort) {
		isNumericSort = isNumericSort || false; // by default text sort
		let sortable = [];
		for (let key in obj) {
			if (obj.hasOwnProperty(key)) sortable.push([key, obj[key]]);
		}
		if (isNumericSort) {
			sortable.sort(function (a, b) {
				return a[1] - b[1];
			});
		} else {
			sortable.sort(function (a, b) {
				var x = a[1].toLowerCase(),
					y = b[1].toLowerCase();
				return x < y ? -1 : x > y ? 1 : 0;
			});
		}
		return sortable; // array in format [ [ key1, val1 ], [ key2, val2 ], ... ]
	}
};