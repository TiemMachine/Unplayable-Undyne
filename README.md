const uf_ver_string = "UF 1.0";
var uf_debug = false;

// ---- Introduction
// Based on actual fairdyne source
Scene.prototype.selectScene = function(name, data) {

	this.scene_state = name;
	this.scene_time = 0;

	switch(this.scene_state) {
		case "splash":
			break;
		case "gameplay":
            undyne.queue_text([
				{ text: "Are you sure you'd\nlike to play this?" },
				{ text: "Things might get a\nwee bit dizzy..." },
                { text: "This is literally unplayable." },
                { text: "..." },                
                { text: "Okay, here we go!" },
			], uf_begin_game);
			gameplay_stage.alpha = 0;
			break;
	}

}

function uf_begin_game() {
    menu = new Menu();
    menu.select_text.text = "Select an unfairness level."
    menu.options = ["hard", "genocide", "aprilfools"];
    menu.normal_text_text = "I want to get dizzy.";
    menu.hard_text_text = "I want to lessen my eyesight.";
    menu.genocide_text_text = "U SPIN ME RIGHT ROUND ROUND";
    menu.show();
    menu.updateLove();
}

Menu.prototype.updateLove = function() {
	heart.setMaxHP(4);
}

const uf_undyne_responses = ["Please don't\nconcentrate too hard.",
                             "You know, Gerson\nsells some glasses.\nMaybe that can help\nyou.",
                             "I am not responsible\nfor any accidents\nthatmight follow."]

Menu.prototype.select = function() {
	this.hide();
	se_menu_select.play();
	undyne.queue_text([{text: uf_undyne_responses[this.current_option]}], gamestate.restartGame.bind(gamestate, this.options[this.current_option]));
}

// ---- Tools

function uf_make_invulnerable() {
	setInterval(function() { heart.invincibility = 1000; }, 250);
}

// ---- Signals

Heart.prototype._uu_takeDamage = Heart.prototype.takeDamage;

Heart.prototype.takeDamage = function (damage) {
    if (this.invincibility > 0) return;
    console.warn("UnfairFrisk: took damage!");
    this._uu_takeDamage(damage);
}

// ---- Arrows and shield attack, plus rotating heart

const uf_arrow_dir_names = ["???", "down", "left", "up", "right"];
const uf_heart_dir_names = ["!!!", "right", "down", "left", "up"];
const uf_turntype_names = ["straight", "Xclock", "revs", "revs", "clock"];

Arrow.prototype._uu_update = Arrow.prototype.update;
Heart.prototype._uu_update = Heart.prototype.update;

Arrow.prototype.update = function (delta_ms) {
	this._uu_update(delta_ms);
    this.sprite.rotation += 3 / (delta_ms * 8) * Math.PI;
    this.sprite.rotation %= 2 * Math.PI;
}

Heart.prototype.update = function (delta_ms) {
    var rotation = this.shield_sprite.rotation;
    this._uu_update(delta_ms);
    this.shield_sprite.rotation += 27 / (delta_ms * 8) * Math.PI;
    this.shield_sprite.rotation %= 2 * Math.PI;

    // Rotating heart
    this.sprite.rotation += 27 / (delta_ms * 8) * Math.PI;
    this.sprite.rotation %= 2 * Math.PI;
}

// ---- Pike attack (from top and bottom)

Pike.prototype._uu_update = Pike.prototype.update;

var pikes_in_cols = [undefined, 0, 0, 0];
var x_for_cols = [undefined, 299, 320, 341];

Pike.prototype.update = function (delta_ms) {
    var recalc_dodge = false;

    this.active_time += delta_ms;
    this.sprite.rotation += 27 / (delta_ms * 8) * Math.PI;
    this.sprite.rotation %= 2 * Math.PI;
    if (this.active_time >= this.appear_time && !this.shot) {
        pikes_in_cols[this.initial_position] += 1;
        console.log("added pike from " + this.initial_position + ":   " + pikes_in_cols);
        recalc_dodge = true;
    }

    this._uu_update(0); // active_time was updated earlier

    if ((this.active_time >= this.appear_time + 150 || this.collidesWithHeart() || this.removed || this.active_time > spear_total_time) && !this.uf_removed) {
        this.uf_removed = true;
        pikes_in_cols[this.initial_position] -= 1;
        console.log("removed pike from " + this.initial_position + ": " + pikes_in_cols);
        recalc_dodge = true;
    }
    
    // bug preventer!
    if (pikes_in_cols[1] < 0) pikes_in_cols[1] = 0;
    if (pikes_in_cols[2] < 0) pikes_in_cols[2] = 0;
    if (pikes_in_cols[3] < 0) pikes_in_cols[3] = 0;

    if (recalc_dodge) uf_dodge_pikes(this.initial_position);
}

// ---- Spears attack (sides)

Spear.prototype._uu_update = Spear.prototype.update;

function uf_getdir() {
    var rbool = Math.round(Math.random()) == 0;
    return rbool ? 1 : -1;
}

Spear.prototype.update = function(delta_ms) {
    this.sprite.rotation += 27 / (delta_ms * 8) * Math.PI;
    this.sprite.rotation %= 2 * Math.PI;
    if (this.shot) {
        var dir_x = uf_getdir(), dir_y = uf_getdir();
        
		this.pos_x += this.direction.x * SPEAR_SPEED * delta_ms;
		this.pos_y += this.direction.y * SPEAR_SPEED * delta_ms;
        this.pos_x -= this.direction.x * SPEAR_SPEED * delta_ms;
        this.pos_y -= this.direction.y * SPEAR_SPEED * delta_ms;
    }

    this._uu_update(delta_ms);
}
